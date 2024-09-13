# PEDIDO_TARJETA
## ms_Pedido_Tarjeta_Novopayment
1. Recibe un **XML Profile** llamado `Perfil_Detalle_Pedido_Tarjeta` que pasa por un **MAP** llamado `Transformacion_Perfil_Detalle_Pedido_Tarjeta_to_Perfil_Request_Pedido_Tarjeta_Lista_Novopayment` y se transforma en un **JSON Profile** llamado `Perfil_Request_Pedido_Tarjeta_Lista_Novopayment`.
2. Entra en un **BRANCH SHAPE** de 3 ramas: 
    - 2.1 Entra a un cache llamado ``Cache_Perfil_Detalle_Pedido_Tarjeta``, que tiene como **Profile XML**  llamado ``Perfil_Detalle_Pedido_Tarjeta`` , que es identificado por el **IdPedidoTarjeta**, entonces pide que se elimine del cache.
    - 2.2 Luego entra por otro **MAP** de datos llamado ``Transformacion_Perfil_Detalle_Pedido_Tarjeta_to_Perfil_Detalle_Pedido_Tarjeta_TRXID`` que recibe un **JSON Profile** ``Perfil_Request_Pedido_Tarjeta_Lista_Novopayment`` y se convierte denuevo a **XML Profile** ``Perfil_Detalle_Pedido_Tarjeta``
    - 3.3 Y por ultimo entra a un **Process Route** llamado ``pr_EnrutaConector`` que  llama al proceso ``CONECTOR_NOVOPAYMENT``.
    Ese proceso al terminar de ejecutarse, entra por un **Notify Shape** llamado `RESPONSE_NOVO_EMISION`:
        Message
        ```
            JsonResponse:
                {1}
                HTTP Status: 
                {2}
                HTTP Message:
                {3}
        ```

        Variables
        ```	
            {1}	Current Data
            {2}	Document Property - Base - Application Status Code
            {3}	Document Property - Base - Application Status Message

        ```
        Luego entra en una **Decision Shape** llamada `Respuesta Servicio?` que valida si el `Document Property - Base - Application Status Code` es igual `200`.
        - Si es verdadero = `200`, entra por otra **Decision Shape** llamado `V Error Desconocido` que valida si el ``first Value`` de tipo **JSON** llamado `Perfil_Error_Generico_Novopayment` el `rc` es igual a vacio o no.
            - Si es `` NO VACIO`` entra por un **MAPS** llamado `Transformacion_Perfil_Response_Pedido_Tarjeta_Lista_Novopayment_to_Perfil_Response_Pedido_Tarjeta_Lista_Novopayment_XML` que convierte un **JSON Profile** `Perfil_Response_Pedido_Tarjeta_Lista_Novopayment` a **XML Profile** `Perfil_Response_Pedido_Tarjeta_Lista_Novopayment`.
            Donde a la ves entra por un ultimo **Decision Shape** que valida si el `rc` es igual a `0`. 
                - Si es `0`, entra por un **Branch Shape** de 2 ramas:
                    1. la primera rama hace que agrege al Cache de tipo **Profile XML** `Cache_Perfil_Response_Pedido_Tarjeta_Lista_Novopayment` que tiene como ID `TrxId`.
                    2. Y la segunda Rama hace que retorne un `OK Novopayment`.
                - Pero si no es `0`, crea un **Message Shape** llamado `Perfil_Novopayment_Error`:
                    Message
                    ```
                    <RESPONSE_PEDIDO_TARJETA_LIST_NOVOPAYMENT>
                        <rc>{1}</rc>
                        <msg>{2}</msg>
                        <trxId>{3}</trxId>
                        <transactionDate></transactionDate>
                        <identity>{4}</identity>
                    </RESPONSE_PEDIDO_TARJETA_LIST_NOVOPAYMENT>
                    ```
                    Variables

                    `{1}	XML Profile - Perfil_Response_Pedido_Tarjeta_Lista_Novopayment - rc (RESPONSE_PEDIDO_TARJETA_LIST_NOVOPAYMENT/rc)`
                    `{2}	XML Profile - Perfil_Response_Pedido_Tarjeta_Lista_Novopayment - msg (RESPONSE_PEDIDO_TARJETA_LIST_NOVOPAYMENT/msg)`
                    `{3}	Document Cache Lookup`
                    `{4}	XML Profile - Perfil_Response_Pedido_Tarjeta_Lista_Novopayment - identity (RESPONSE_PEDIDO_TARJETA_LIST_NOVOPAYMENT/identity)`

                    Entra por **Branch Shape** de 2 ramas:
                    - Agrega al Cache `Cache_Perfil_Response_Pedido_Tarjeta_Lista_Novopayment`
                    - Retorna un `FALSE`. 
            - Y si es `Vacio`, entra en un **MAPS** llamado `Transformacion_Perfil_Sin_Datos_To_Perfil_Response_Pedido_Tarjeta_Lista_Novopayment` que convierte un **JSON Profile** `Perfil_Error_Generico_Novopayment` a **XML Profile** `Perfil_Response_Pedido_Tarjeta_Lista_Novopayment`.
            Luego crea un **Message Shape** `Perfil_Novopayment_Error`:
                Message
                ```
                <RESPONSE_PEDIDO_TARJETA_LIST_NOVOPAYMENT>
                    <rc>{1}</rc>
                    <msg>{2}</msg>
                    <trxId>{3}</trxId>
                    <transactionDate></transactionDate>
                    <identity>{4}</identity>
                </RESPONSE_PEDIDO_TARJETA_LIST_NOVOPAYMENT>
                ```
                Variables
                `{1}	XML Profile - Perfil_Response_Pedido_Tarjeta_Lista_Novopayment - rc (RESPONSE_PEDIDO_TARJETA_LIST_NOVOPAYMENT/rc)`
                `{2}	XML Profile - Perfil_Response_Pedido_Tarjeta_Lista_Novopayment - msg (RESPONSE_PEDIDO_TARJETA_LIST_NOVOPAYMENT/msg)`
                `{3}	Document Cache Lookup`
                `{4}	XML Profile - Perfil_Response_Pedido_Tarjeta_Lista_Novopayment - identity (RESPONSE_PEDIDO_TARJETA_LIST_NOVOPAYMENT/identity)`

                Entra por **Branch Shape** de 2 ramas:
                    - Agrega al Cache `Cache_Perfil_Response_Pedido_Tarjeta_Lista_Novopayment`
                    - Retorna un `FALSE`.
        - Si la respuesta no es `200` entra en un **MAPS** llamado `Transformacion_Response_Perfil_Sin_Datos_To_Perfil_Response_Pedido_Tarjeta_Lista_Novopayment` que convierte un **Flat File Property** `Perfil_Sin_Datos` a **XML Profile** `Perfil_Sin_Datos`.
        Entra por una **Decision Shape** `HTTP Status -1?` que valida si `Document Property - Base - Application Status Code` es igual a `-1`.
            - ambas situaciones hace un **Message Shape** `Perfil_Novopayment_Error`:
                Message
                ```
                <RESPONSE_PEDIDO_TARJETA_LIST_NOVOPAYMENT>
                    <rc>{1}</rc>
                    <msg>{2}</msg>
                    <trxId>{3}</trxId>                   
                    <identity>{4}</identity>
                    <transactionDate></transactionDate>
                </RESPONSE_PEDIDO_TARJETA_LIST_NOVOPAYMENT>
                ```
                Variables
                `{1}	XML Profile - Perfil_Response_Pedido_Tarjeta_Lista_Novopayment - rc (RESPONSE_PEDIDO_TARJETA_LIST_NOVOPAYMENT/rc)`
                `{2}	XML Profile - Perfil_Response_Pedido_Tarjeta_Lista_Novopayment - msg (RESPONSE_PEDIDO_TARJETA_LIST_NOVOPAYMENT/msg)`
                `{3}	Document Cache Lookup`
                `{4}	XML Profile - Perfil_Response_Pedido_Tarjeta_Lista_Novopayment - identity (RESPONSE_PEDIDO_TARJETA_LIST_NOVOPAYMENT/identity)`

                Entra por **Branch Shape** de 2 ramas:
                - Agrega al Cache `Cache_Perfil_Response_Pedido_Tarjeta_Lista_Novopayment`
                - Retorna un `FALSE`.

## ms_Pedido_Tarjeta_Novopayment_ACTUALIZA_ESTADO_PEDIDO

Recibe un **XML Profile** `Perfil_Detalle_Pedido_Tarjeta` que pasa por un **MAPS** que devuelve un **JSON Profile** `Perfil_Request_Pedido_Tarjeta_Lista_Novopayment`, se setean 2 propiedades que son:
1.  `Value_Path_Operation` = `/sodexo_issuanceList?trxId=`
2. `DDP_REQUEST_JSON` = `Current Data`

Entra por un **Branch Shape** de 3 ramas:
1. Elimina del Cache `Cache_Perfil_Detalle_Pedido_Tarjeta` 
2. Pasa por un **MAPS** `Transformacion_Perfil_Detalle_Pedido_Tarjeta_to_Perfil_Detalle_Pedido_Tarjeta_TRXID` de un **JSON Profile** `Perfil_Request_Pedido_Tarjeta_Lista_Novopayment` a **XML Profile**
`Perfil_Detalle_Pedido_Tarjeta`. 
Luego Agrega al mismo Cache `Cache_Perfil_Detalle_Pedido_Tarjeta`
3. Se conecta a un **Process Route** `pr_EnrutaConector` que llama al proceso `CONECTOR_NOVOPAYMENT` y la respuesta pasa por un **Notify Shape** `RESPONSE_NOVO_EMISION`:

    Message
    ```
        JsonResponse:
            {1}
        HTTP Status: 
            {2}
        HTTP Message:
            {3}
    ```
    Variables
    ```	
        {1}	Current Data
        {2}	Document Property - Base - Application Status Code
        {3}	Document Property - Base - Application Status Message

     ```
    Luego se setea un `ID_RECORD` = `0`
    Luego entra en una **Decision Shape** llamada `Respuesta Servicio?` que valida si el `Document Property - Base - Application Status Code` es igual `200`.
    - Si es verdadero = `200`, entra por otra **Decision Shape** llamado `V Error Desconocido` que valida si el ``first Value`` de tipo **JSON** llamado `Perfil_Error_Generico_Novopayment` el `rc` es igual a vacio o no.
        - Si es `` NO VACIO`` entra por un **MAPS** llamado `Transformacion_Perfil_Response_Pedido_Tarjeta_Lista_Novopayment_to_Perfil_Response_Pedido_Tarjeta_Lista_Novopayment_XML` que convierte un **JSON Profile** `Perfil_Response_Pedido_Tarjeta_Lista_Novopayment` a **XML Profile** `Perfil_Response_Pedido_Tarjeta_Lista_Novopayment`.
            Donde a la ves entra por un ultimo **Decision Shape** que valida si el `rc` es igual a `0`. 
            - Si es `0`, entra por un **Branch Shape** de 2 ramas:
                    1. la primera rama hace que agrege al Cache de tipo **Profile XML** `Cache_Perfil_Response_Pedido_Tarjeta_Lista_Novopayment` que tiene como ID `TrxId`.
                    2. Y la segunda Rama hace que retorne un `OK Novopayment`.
            - Pero si no es `0`, crea un **Message Shape** llamado `Perfil_Novopayment_Error`:
                Message
                ```
                <RESPONSE_PEDIDO_TARJETA_LIST_NOVOPAYMENT>
                     <rc>{1}</rc>
                    <msg>{2}</msg>
                    <trxId>{3}</trxId>
                    <transactionDate></transactionDate>
                     <identity>{4}</identity>
                </RESPONSE_PEDIDO_TARJETA_LIST_NOVOPAYMENT>
                ```
                Variables

                `{1}	XML Profile - Perfil_Response_Pedido_Tarjeta_Lista_Novopayment - rc (RESPONSE_PEDIDO_TARJETA_LIST_NOVOPAYMENT/rc)`
                `{2}	XML Profile - Perfil_Response_Pedido_Tarjeta_Lista_Novopayment - msg (RESPONSE_PEDIDO_TARJETA_LIST_NOVOPAYMENT/msg)`
                `{3}	Document Cache Lookup`
                `{4}	XML Profile - Perfil_Response_Pedido_Tarjeta_Lista_Novopayment - identity (RESPONSE_PEDIDO_TARJETA_LIST_NOVOPAYMENT/identity)`

                Entra por **Branch Shape** de 2 ramas:
                - Agrega al Cache `Cache_Perfil_Response_Pedido_Tarjeta_Lista_Novopayment`
                - Retorna un `FALSE`. 
            - Y si es `Vacio`, entra en un **MAPS** llamado `Transformacion_Perfil_Sin_Datos_To_Perfil_Response_Pedido_Tarjeta_Lista_Novopayment` que convierte un **JSON Profile** `Perfil_Error_Generico_Novopayment` a **XML Profile** `Perfil_Response_Pedido_Tarjeta_Lista_Novopayment`.
            Luego crea un **Message Shape** `Perfil_Novopayment_Error`:
                Message
                ```
                <RESPONSE_PEDIDO_TARJETA_LIST_NOVOPAYMENT>
                    <rc>{1}</rc>
                    <msg>{2}</msg>
                    <trxId>{3}</trxId>
                    <transactionDate></transactionDate>
                    <identity>{4}</identity>
                </RESPONSE_PEDIDO_TARJETA_LIST_NOVOPAYMENT>
                ```
                Variables
                `{1}	XML Profile - Perfil_Response_Pedido_Tarjeta_Lista_Novopayment - rc (RESPONSE_PEDIDO_TARJETA_LIST_NOVOPAYMENT/rc)`
                `{2}	XML Profile - Perfil_Response_Pedido_Tarjeta_Lista_Novopayment - msg (RESPONSE_PEDIDO_TARJETA_LIST_NOVOPAYMENT/msg)`
                `{3}	Document Cache Lookup`
                `{4}	XML Profile - Perfil_Response_Pedido_Tarjeta_Lista_Novopayment - identity (RESPONSE_PEDIDO_TARJETA_LIST_NOVOPAYMENT/identity)`

                Entra por **Branch Shape** de 2 ramas:
                    - Agrega al Cache `Cache_Perfil_Response_Pedido_Tarjeta_Lista_Novopayment`
                    - Retorna un `FALSE`.
        - Si la respuesta no es `200` entra en un **MAPS** llamado `Transformacion_Response_Perfil_Sin_Datos_To_Perfil_Response_Pedido_Tarjeta_Lista_Novopayment` que convierte un **Flat File Property** `Perfil_Sin_Datos` a **XML Profile** `Perfil_Sin_Datos`.
        Entra por una **Decision Shape** `HTTP Status -1?` que valida si `Document Property - Base - Application Status Code` es igual a `-1`.
            - ambas situaciones hace un **Message Shape** `Perfil_Novopayment_Error`:
                Message
                ```
                <RESPONSE_PEDIDO_TARJETA_LIST_NOVOPAYMENT>
                    <rc>{1}</rc>
                    <msg>{2}</msg>
                    <trxId>{3}</trxId>                   
                    <identity>{4}</identity>
                    <transactionDate></transactionDate>
                </RESPONSE_PEDIDO_TARJETA_LIST_NOVOPAYMENT>
                ```
                Variables
                `{1}	XML Profile - Perfil_Response_Pedido_Tarjeta_Lista_Novopayment - rc (RESPONSE_PEDIDO_TARJETA_LIST_NOVOPAYMENT/rc)`
                `{2}	XML Profile - Perfil_Response_Pedido_Tarjeta_Lista_Novopayment - msg (RESPONSE_PEDIDO_TARJETA_LIST_NOVOPAYMENT/msg)`
                `{3}	Document Cache Lookup`
                `{4}	XML Profile - Perfil_Response_Pedido_Tarjeta_Lista_Novopayment - identity (RESPONSE_PEDIDO_TARJETA_LIST_NOVOPAYMENT/identity)`

                Entra por **Branch Shape** de 2 ramas:
                - Agrega al Cache `Cache_Perfil_Response_Pedido_Tarjeta_Lista_Novopayment`
                - Retorna un `FALSE`.

## ms_SP_INSERT_WSTRANSAPENDIENTE

Inicia rebiendo datos, luego hace un seteo `ID_RECORD` = `0`, luego pasa por un **MAPS** `Transformacion_Perfil_Parametros_Entrada_Generico_Salida_ms_SP_INSERT_WSTRANSAPENDIENTE_XML` que convierte un **XML Profile** `Perfil_Parametros_Entrada_Generico_Salida` en un **XML Profile** `Perfil_Parametros_Entrada_Generico_Salida`. Entra en un **Branch Shape** de 2 ramas:
1. Agrega al Cache `Cache_Perfil_Parametros_Entrada_Generico_Salida`
2. Hace una coneccion `Conexion_DB_CONDOR` y realiza la operacion `Operacion_SP_INSERT_WSTRANSAPENDIENTE` y envia como parametros :

    ```
        Nombre : P_METODOWS, XML Profile : Perfil_Parametros_Entrada_Generico_Salida , Valor : PARAMETRO2
        Nombre : P_IDPRODUCTO, XML Profile : Perfil_Parametros_Entrada_Generico_Salida , Valor : PARAMETRO3
        Nombre : P_TRXID, XML Profile : Perfil_Parametros_Entrada_Generico_Salida , Valor : PARAMETRO4
        Nombre : P_NRODOCUMENTO, XML Profile : Perfil_Parametros_Entrada_Generico_Salida , Valor : PARAMETRO5
        Nombre : P_JSON_REQUEST, XML Profile : Perfil_Parametros_Entrada_Generico_Salida , Valor : PARAMETRO6
    ```
Luego setea nuevamente `ID_RECORD` = `0`, conecta con un **MAPS** `Transformacion_Perfil_SP_INSERT_WSTRANSAPENDIENTE_to_Perfil_Parametros_Entrada_Generico_Salida` que convierte un **Database Profile** `Perfil_SP_INSERT_WSTRANSAPENDIENTE` a un **XML Profile** `Perfil_Parametros_Entrada_Generico_Salida`.
Continua con un **Message Shape** `Respuesta exitosa` :
Message
```
<PARAMETROS_SP_GENERICO>
    <PARAMETRO1>{1}</PARAMETRO1>
    <PARAMETRO2>{2}</PARAMETRO2>
</PARAMETROS_SP_GENERICO>
``` 
Variables
```
{1}	Document Cache Lookup
{2}	XML Profile - Perfil_Parametros_Entrada_Generico_Salida - PARAMETRO2 (PARAMETROS_SP_GENERICO/PARAMETRO2)

```
Luego se Conecta a un **Branch Shape** de 2 ramas:
1. Elimina del Cache `Cache_Perfil_Parametros_Entrada_Generico_Salida`
2. Entra en una **Decision Shape** `V Error SP` y compara el `PARAMETRO2` = `1`.
    - Si es verdadero, retorna un `OK`.
    - Si es Falso retorna un `FALSE`.

