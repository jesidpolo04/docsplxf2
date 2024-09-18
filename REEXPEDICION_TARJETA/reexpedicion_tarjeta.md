# RETIRO_BENEFICIARIOREEXPEDICION_TARJETA

## job_Reexpide_Tarjeta

Frecuencia de ejecucion: Programado

### Sistemas involucrados: 

- Novopayment (operacion: /sodexo_replaceCard)
- Condor BD Oracle


### Descripcion general:
Proceso Job sincronico ejecutado para reexpedir tarjetas.   



El proceso inicia cuando un scheduler lanza el Job job_Reexpide_Tarjeta. Para ello se ejecuta en Condor BD `SP_GET_SOLICITUD_REEXPEDICION` y si se cumplen las validaciones iniciales, se lanza el subproceso principal `ri_Reexpide_Tarjeta`


### Actividades del proceso: 
Subproceso principal: `ri_Reexpide_Tarjeta`

![Proceso](assets/ri_Reexpide_Tarjeta.png)





