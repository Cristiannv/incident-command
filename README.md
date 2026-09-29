# Incident Command

Prototipo estático para correlacionar señales sintéticas de observabilidad, estimar impacto y proponer una acción segura.

## Demo local

Abre [`index.html`](./index.html) directamente en un navegador moderno. No requiere dependencias, servidor ni conexión de red. La interfaz es responsive; permite filtrar eventos por tipo y seleccionar una señal para leer su contexto.

## Lectura del incidente

La señal más directa es `Cannot find export 'normalizeCallIndex'` en `call-index.js`, seguida por reinicios de systemd. En la misma ventana, el health check de `live-call-server` falla y frena la versión .5, aunque `preflight` había pasado. El `/health` posterior contesta 200 a través del proxy, pero no demuestra que el servicio de llamadas o WebSockets esté sano.

**Hipótesis:** una incompatibilidad de export/import pudo causar la degradación de llamadas. Confianza media: el log de runtime no identifica el build que ejecutó el import y el deploy indica que la versión .5 fue detenida.

**Impacto observable:** 3 de 20 seats tienen heartbeat atrasado; S003 y S007 además aparecen `unavailable` por WebSocket. S014 tuvo un timeout de reconexión previo. Hay 3 llamadas activas en el snapshot, sin overlap con esos seats; los datos no permiten contar personas afectadas ni demostrar que esas llamadas siguieron sanas.

Los `401 session_expired` de Beta contrastan con quotes 200 de Alpha y empiezan antes del error de import. Se presentan como una señal posiblemente separada hasta correlacionar más datos.

**Siguiente paso:** conservar el build estable; verificar el export y el build realmente activo; reproducir import, health check específico y flujo de llamada en staging; luego canary con observabilidad y rollback definido.

