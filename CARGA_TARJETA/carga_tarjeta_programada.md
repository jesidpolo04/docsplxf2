# Carga Programada Tarjeta

## Job_Carga_programada

Frecuencia de ejecucion: Cada 30 minutos

### Sistemas involucrados: 

 - Condor DB Oracle

 - Novopayment API: https://tomcat-uat-sodexo.novopayment.net/sodexoapi/1.0/{2}/employee/{1}{4}{3}
	
    `{1}` Value_employee
	
    `{2}` value_programId
	
    `{3}` value_trxid
	
    `{4}` value_path_operation

### Descripcion general:
Proceso de integracion entre condor y novo para realizar las cargas masivas de dinero a las cuentas (tarjetas) de clientes organizadas por pedidos. 

Existe un job llamado en boomi `Job_Carga_programada` que corre cada 30 minutos.

Condor se encarga de las validaciones a los pedidos antes de dejarlos listo para el job de boomi.

Un job puede procesar varios pedidos (los que esten programados, pendiente validar volumetria).

Un pedido puede contener hasta `2000` cuentas, una cuenta solo pertenece a un producto (Canasta, Dotacion, etc), un pedido solo tiene una empresa (cliente).

Los pedidos no se pueden modificar. Siempre se manda a Novo el `idCuenta` original del cliente (Si la cuenta no tiene tarjeta activa, se carga a la cuenta). Si en Novo se procesa el pedido, no se puede modificar ni volver a enviar, no se puede enviar otro pedido porque rompe la relacion pedido-factura.

Si quedan cuentas (tarjetas) pendientes, se deben manejar como `traslados` (novedad monetaria)

### Variables globales:
- TIPO_TRANSACCION
- DPP_NOMBRE_PROCESO
- OPERACION

### Actividades del proceso: 
Subproceso: `ri_Cargas_Programadas_NovoPayment_Condor`

1. Consulta Productos en `Condor DB` -> Conector: `Operacion_Obtener_Productos_NovoPayment` -> Mapeo `Transformacion_Productos_Novopayment` -> Resultado `Producto_cargado`

<!-- EJEMPLO.... -->

2. Guardar Cache Productos_Novopayment

<!--VALIDAR CON EJEMPLOS-->
3. Por cada producto cargado - PEDIDO:

    Ejecutar `ms_recuperar_carga_programada`: 

    Se encarga de ejecutar en Condor DB `SP_RECARGASPROGRAMADAS` 
    
    Mapeo SP_RECARGASPROGRAMADAS_to_XML 	
	
    Establecer variable `DPP_NUMERO_PEDIDO`	
    
    Ejecutar Condicional:
	    
        if	TIPO_TRANSACCION = G
		    set TIPO_TRANSACCION = G
     
    Establecer perfil:
    ```xml
        <PRODUCTOS_CARGADOS>
        <IDPRODUCTO>Objeto `Pefil_Datos_Tarjeta`</IDPRODUCTO>
        </PRODUCTOS_CARGADOS>	
    ```
    Guardar `Cache_Productos_cargados`
	
	Mapeo `Transformacion_Datos_Tarjeta_Datos_Tarjeta_Detalle`: `Perfil_Datos_tarjeta_detalle`

	Guardar `cache_Datos_tarjeta_detalle`
	
	Mapeo `Transformacion_Cache_Encabezado`: `Perfil_Datos_tarjeta_Encabezado_individual`
	
    Guardar `Cache_Datos_Tarjeta_Encabezado`

	Mapeo `Transformacion_Datos_tarjeta_encabezado`: `perfil_datos_tarjeta_encabezado` 
	
	Set `NroDocumento`, `Cod_producto`, `idProducto`
	
	Mapeo `Transformacion_Datos_Pedido_NovoPayment`: `Perfil_datos_pedido_novopayment`
	
	Ejecuta `ms_Carga_Programadas_NovoPayment` con entrada: `perfi_datos_pedido_novopayment`

	Mapeo a `Perfil_Request_Pedido_Novopayment`

	Ejecutar subproceso `ConnectorHttpClientNovopayment` (este subproceso se conecta a Novopayment API por HTTPS):

		#Ejecuta script personalizado (groovy 2.4) para validar token
		import java.util.Properties;
        import java.io.InputStream;
        import java.text.SimpleDateFormat
        import java.util.Calendar
        import java.util.Date
        import com.boomi.execution.ExecutionUtil;
        // Se recuperan valores de propiedades
        def MinutosVigenciaToken = ExecutionUtil.getProcessProperty("2bf59542-f315-4d8a-a662-7292e4a12aeb", "5012033f-95ad-4371-8e34-1113f4bf2e51");
        def FechaGeneracionToken = ExecutionUtil.getProcessProperty("2bf59542-f315-4d8a-a662-7292e4a12aeb", "de221a1a-b89a-4fde-a2cf-7046cb2efec3");
        def FechaActual = new Date()


        // Se adicionan los minutos a la fecha de generación del token

        def formatoEntrada = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss")


        try {
            // Intenta parsear la cadena de texto a una fecha
            def fecha = formatoEntrada.parse(FechaGeneracionToken)
            
            
            def calendar = Calendar.getInstance()
            calendar.time = fecha
            

            calendar.add(Calendar.MINUTE, MinutosVigenciaToken.toInteger())
            def nuevaFecha = calendar.time
            
            // Se valida vencimiento de token
            if (nuevaFecha.before(FechaActual))
            {
                // Se refresca access token para que utilice el refresh token

                ExecutionUtil.setProcessProperty("2bf59542-f315-4d8a-a662-7292e4a12aeb", "88eb81b6-46eb-4c65-ae9d-94f399ff9c7b", "");
                println("token vencido")
            
            }
            else{
                println("token valido")
            }
            
        } catch (Exception e) {
            // En caso de que la cadena no sea válida como fecha
            // Se refresca access token para que utilice el refresh token por error
            ExecutionUtil.setProcessProperty("2bf59542-f315-4d8a-a662-7292e4a12aeb", "88eb81b6-46eb-4c65-ae9d-94f399ff9c7b", "");
            println("La cadena no es una fecha válida o int valido.")
        }


        for( int i = 0; i < dataContext.getDataCount(); i++ ) {
            InputStream is = dataContext.getStream(i);
            Properties props = dataContext.getProperties(i);

            dataContext.storeStream(is, props);
        }

    Si no tiene token: Ejecuta API Novopayment POST: `OperacionConnectorNovopaymentAuth` (perfil entrada json)

	Si tiene token vencido, refrescar token ejecuta API Novopayment POST: `OperacionConnectorNovopaymentAuthRefresh` (perfil entrada json)  
	
    Si no funciona el bloque anterior de autenticacion, reintenta

	Luego de autenticarse, se arma mensaje json para llamar al servicio Novo:

    <!-- validar perfil de entrada en la operacion novo -->

	    Si Value_employee es igual a vacio:
			Ejecuta OperacionConectorAPINovopaymentExcluyeEmployee
		En caso contrario:
			Ejecuta API Novopayment POST: OperacionConnectorAPINovopayment (perfil entrada json )
	
    Resultado: `Perfil_response_pedido_novopayment`
			
	Si se ejecuta correctamente: actualiza `Trx_id` y `orderId` (nro de pedido en condor) ejecutando en condor db: `Operacion_SP_UPDATE_PEDIDOCARGA_TRXID`
	
	Registra log en Condor DB de todo el proceso ya sea errores o pasos exitosos a traves del llamado al Web service de Logs en Condor 

