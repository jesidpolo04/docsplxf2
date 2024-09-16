## job_SP_GENERAR_EMISION
Inicia el proceso, luego entra en un **Branch Process** de 2 ramas:
1. Crea un **Message Shape** `Construcción Perfil SP_INSERT_WSLOGPETICION`:
    ```
    <root>
	    <PARAMETROS_SP_INSERT_WSLOGPETICION>
            <P_METODOWS>pedidoTarjetas</P_METODOWS>
            <P_IDPRODUCTO></P_IDPRODUCTO>
            <P_TRXID></P_TRXID>
            <P_CODIGORESPUESTA>200</P_CODIGORESPUESTA>
            <P_DESCRIPCIONRESPUESTA>Proceso iniciado job_SP_GENERAR_EMISION</P_DESCRIPCIONRESPUESTA>
            <P_HTTPRESPONSE></P_HTTPRESPONSE>
            <P_IDUSUARIOAPP>busservicios</P_IDUSUARIOAPP>
            <P_USUARIO>busservicios</P_USUARIO>
	    </PARAMETROS_SP_INSERT_WSLOGPETICION>
    </root>
    ```
    Luego entra a un **Process Route** `Proceso_Enruta_Procesos` y entra al proceso `ms_Registro_Log` y termina la rama.
2. Entra a un **Conector Shape**,  una coneccion llamada `Conexion_DB_CONDOR` y realiza la operacion `Operacion_SP_GENERAR_EMISION`.
Luego se conecta a un **Branch Shape** de 2 ramas :
    1. Crea un **Message Shape** `Construcción Perfil                   SP_INSERT_WSLOGPETICION`:
        ```
        <root>
	        <PARAMETROS_SP_INSERT_WSLOGPETICION>
                <P_METODOWS>pedidoTarjetas</P_METODOWS>
                <P_IDPRODUCTO></P_IDPRODUCTO>
                <P_TRXID></P_TRXID>
                <P_CODIGORESPUESTA>200</P_CODIGORESPUESTA>
                <P_DESCRIPCIONRESPUESTA>Proceso iniciado job_SP_GENERAR_EMISION</P_DESCRIPCIONRESPUESTA>
                <P_HTTPRESPONSE></P_HTTPRESPONSE>
                <P_IDUSUARIOAPP>busservicios</P_IDUSUARIOAPP>
                <P_USUARIO>busservicios</P_USUARIO>
	        </PARAMETROS_SP_INSERT_WSLOGPETICION>
        </root>
         ```
        Luego entra a un **Process Route** `Proceso_Enruta_Procesos` y entra al proceso `ms_Registro_Log` y termina la rama.
    2. Entra a otro **Message Shape** `Mensaje_Cola`:
    Mensaje
        `Ejecución {1}`
    Variables
        `{1}	Date/Time - Current Date - yyyy-MM-dd'T'HH:mm:ss.SSS'Z'`
    Y se conecta a un **Conector Shape** de tipo `Boomi Atom Queue`, con coneccion llamado `Conexion_Cola_Pedido_Tarjeta_Masivo` y ejecuta la operacion `Cola_Escribe_Pedido_Tarjeta_Masivo`.

## queue_Pedido_Tarjeta_Condor_Archivo_NOPRD
Inicia con un **Start Shape** de conector a `Boomi Atom Queue` , con la coneccion `Conexion_Cola_Pedido_Tarjeta_Condor_Archivo` y ejecutando la operacion `Operacion_Escucha_Cola_Pedido_Tarjeta_Condor_Archivo`.
Luego se conecta a un **Process Route** llamado `Proceso_Enruta_Procesos` y ejecuta el proceso `ri_Pedido_Tarjeta_Condor_Archivo`.
Y termina este proceso.

## queue_SP_GET_PEDIDOTARJETA_JOB
Inicia el proceso con un **Start Shape** de conector `Boomi Atom Queue` con coneccion `Conexion_Cola_Pedido_Tarjeta_Masivo` y la operacion `Cola_Escucha_Pedido_Tarjeta_Masivo`.
Luego entra en un **Branch Shape** de 2 ramas:
1. Entra en un **Message Shape** `Construcción Perfil SP_INSERT_WSLOGPETICION`:
    Message
    ```
    <root>
        <PARAMETROS_SP_INSERT_WSLOGPETICION>
            <P_METODOWS>pedidoTarjetas</P_METODOWS>
            <P_IDPRODUCTO></P_IDPRODUCTO>
            <P_TRXID></P_TRXID>
            <P_CODIGORESPUESTA>200</P_CODIGORESPUESTA>
            <P_DESCRIPCIONRESPUESTA>Proceso iniciado queue_SP_GET_PEDIDOTARJETA_JOB</P_DESCRIPCIONRESPUESTA>
            <P_HTTPRESPONSE></P_HTTPRESPONSE>
            <P_IDUSUARIOAPP>busservicios</P_IDUSUARIOAPP>
            <P_USUARIO>busservicios</P_USUARIO>
        </PARAMETROS_SP_INSERT_WSLOGPETICION>
    </root>
    ```
    Luego entra a un **Process Route** `Proceso_Enruta_Procesos` y llama al proceso `ms_Registro_Log` y termina la rama.

2. Entra en un **Connector Shape** de conector `Database` con coneccion `Conexion_DB_CONDOR` y operacion `Operacion_SP_GET_PEDIDOTARJETA_JOB`
Entra en un **Connector Shape** con conector `Database`, coneccion `Conexion_DB_CONDOR` con la operacion `Operacion_SP_GET_PEDIDOTARJETA_JOB`.
 Luego entra en un **Branch Shape** de 2 ramas:
    1. Entra en un **Message Shape** `Construcción Perfil SP_INSERT_WSLOGPETICION`:
        Message
        ```
        <root>
            <PARAMETROS_SP_INSERT_WSLOGPETICION>
                <P_METODOWS>pedidoTarjetas</P_METODOWS>
                <P_IDPRODUCTO></P_IDPRODUCTO>
                <P_TRXID></P_TRXID>
                <P_CODIGORESPUESTA>200</P_CODIGORESPUESTA>
                <P_DESCRIPCIONRESPUESTA>Proceso iniciado queue_SP_GET_PEDIDOTARJETA_JOB</P_DESCRIPCIONRESPUESTA>
                <P_HTTPRESPONSE></P_HTTPRESPONSE>
                <P_IDUSUARIOAPP>busservicios</P_IDUSUARIOAPP>
                <P_USUARIO>busservicios</P_USUARIO>
            </PARAMETROS_SP_INSERT_WSLOGPETICION>
        </root>
        ```
        Luego entra a un **Process Route** `Proceso_Enruta_Procesos` y llama al proceso `ms_Registro_Log` y termina la rama.
    2. Entra en un **Business Rules Shape** llamado `V ERROR Y CANTIDAD REGISTROS` y ejecuta un **Database Profile** llamado `Perfil_SP_GET_PEDIDOTARJETA_JOB`.
        1. Si es aceptado, entra en un **Maps** `Transformacion_Perfil_SP_GET_PEDIDOTARJETA_JOB_to_Perfil_SP_GET_PEDIDOTARJETA_JOB_XML` que convierte un **Database Profile** `Perfil_SP_GET_PEDIDOTARJETA_JOB` en un **XML Profile** `Perfil_SP_GET_PEDIDOTARJETA_JOB` y entra en un **Process Route** `Proceso_Enruta_Procesos_ReturnData` y ejecuta el proceso grande llamado `ri_Pedido_Tarjeta_Masivo` y termina la rama.
        2. Si no es aceptado, entra en un **Notify Shape** `Error: Recuperando información`:
            Message
            ```
            {1}
            ```
            Variables
            ```           	
            {1}	Document Property - Base - Business Rules Result Message
            ```
            Y termina la rama.

## ws_Pedido_Tarjeta_Condor
Inicia el proceso con un **Start Shape** de conector `Web Services Server`, con accion `Listen` y con operacion `Operacion_Pedido_Tarjeta_Condor`.
Recibe un **JSON Perfil** `Perfil_Request_Pedido_Tarjeta_Condor` y responde un **JSON Profile** `Perfil_Response_Pedido_Tarjeta_Condor`.
Entra en TRY/CATCH:
1. TRY: Entra en un **MAPS** Transformacion_Perfil_Request_Pedido_Tarjeta_Condor_to_Perfil_Request_Pedido_Tarjeta_Condor_XML y convierte el **JSON Profile** `Perfil_Request_Pedido_Tarjeta_Condor` en **XML Profile** `Perfil_Request_Pedido_Tarjeta_Condor`.
Luego se setea un `DPP_INTERFACE` = `1` y entra en un **Process Route** `Proceso_Enruta_Procesos_ReturnData` y ejecuta el proceso `ri_Pedido_Tarjeta_Condor`.
Si todo se ejecuta correcto, entonces entra en **MAPS** `Transformacion_Perfil_Response_Pedido_Tarjeta_Condor_to_Perfil_Response_Pedido_Tarjeta_Condor_JSON` que convierte un **XML Profile** `Perfil_Response_Pedido_Tarjeta_Condor` en **JSON Profile** `Perfil_Response_Pedido_Tarjeta_Condor`.
Luego entra en un **Desicion Shape** `V Error` y valida si el elemento `pSalidaGeneral` es igual a `1`.
    1. Si es correcto, retorna un `True`.
    2. Si es incorrecto, se setea un `outstatuscode` = `400` y retorna `False`.
2. CATCH : crea un **Message Shape** Error Catch:
    Message
    ```
    '{
    "pSalidaGeneral": "1",
    "descripcion": "'{1}'“,
    "codProducto": "'{2}'"
    }
    '
    ```
    Variables
    ```
    {1}	Document Property - Base - Try/Catch Message
    {2}	JSON Profile - Perfil_Request_Pedido_Tarjeta_Condor - codProducto (Root/Object/codProducto)

    ```
    Luego setea un setea un `outstatuscode` = `400` y retorna `False`.

## ws_Pedido_Tarjeta_Condor_Archivo

Inicia el proceso con un **Start Shape** de conector `Web Service Server` y operacion `Operacion_Pedido_Tarjeta_Condor_Archivo` , que recibe un **JSON Profile** `Perfil_Request_Pedido_Tarjeta_Condor_Archivo` y responde otro **JSON Profile** `Perfil_Response_Pedido_Tarjeta_Condor_Archivo`.
Entra en TRY/CATCH:
1. TRY: Entra en un **MAPS** `Transformacion_Perfil_Request_Pedido_Tarjeta_Condor_Archivo_to_Perfil_Request_Pedido_Tarjeta_Condor_Archivo_XML` que mapea del **JSON Profile** `Perfil_Request_Pedido_Tarjeta_Condor_Archivo` en un **XML Profile** `Perfil_Request_Pedido_Tarjeta_Condor_Archivo`.
Luego setea un `DPP_INTERFACE` = 1 y entra en un **Route Process** `Proceso_Enruta_Procesos_ReturnData` y ejecuta el proceso `ri_Pedido_Tarjeta_Condor_Archivo`. Si el proceso falla, termina todo el proceso global.
Pero si el proceso se ejecuta correctamente, entra en un **MAPS** `Transformacion_Perfil_Response_Pedido_Tarjeta_Condor_Archivo_to_Perfil_Response_Pedido_Tarjeta_Condor_Archivo_JSON` Y mapea de un **XML Profile** `Perfil_Response_Pedido_Tarjeta_Condor_Archivo` a un **JSON Profile** `Perfil_Response_Pedido_Tarjeta_Condor_Archivo`.
Luego en un **Desicion Shape**, validamos un paramametro de este perfil de retorno, si `pSalidaGeneral` = `1`:
    1. Si es correcto, retorna un OK y termina todo.
    2. Setea un `outstatuscode` = `400` y retorna un False.
2. CATCH: Crea un **Message Shape** `Error Catch`:
    Message
    ```
    {
    "pSalidaGeneral": 0,
    "descripcion": "'{1}'",
    "idArchivo": "'{2}'",
    "totalBeneficiariosCreado": "0"
    }
    ```
    Variables
    ```   	
    {1}	Document Property - Base - Try/Catch Message
    {2}	JSON Profile - Perfil_Response_Pedido_Tarjeta_Condor_Archivo - idArchivo (Root/Object/idArchivo)   
    ```
    Luego setea un `outstatuscode` = `400` y retorna un False.