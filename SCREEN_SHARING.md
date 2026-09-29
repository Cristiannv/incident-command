# Reto universal: screen sharing sin app

## Respuesta ejecutiva

Un enlace web puede iniciar una sesión de screen sharing de forma razonable en desktop: la persona abre una página HTTPS, pulsa **Compartir pantalla**, el navegador presenta su selector nativo y, tras su elección explícita, la web recibe un `MediaStream` mediante `getDisplayMedia()`. Ese stream puede viajar al agente por WebRTC. La web no debe intentar iniciar la captura al abrir el enlace ni ocultar qué se compartirá.

El requisito estricto —desde el navegador del teléfono, ver la pantalla mientras la persona cambia a cualquier otra app— no es una capacidad web portable. En iPhone Safari y navegadores Android habituales, el soporte web de `getDisplayMedia()` no está disponible de forma interoperable. El sistema operativo sí permite captura desde una app nativa con un diálogo y control explícitos. No hay permiso web que permita a una página observar otras apps en silencio.

## Diferencias por plataforma

| Plataforma | Qué puede hacer el link | Límite principal | Ruta viable para compartir otras apps |
|---|---|---|---|
| Desktop (Chrome, Edge, Firefox y Safari compatibles) | HTTPS + gesto del usuario + selector del navegador; compartir pantalla/ventana/pestaña según navegador. Enviar video por WebRTC. | El navegador decide las fuentes; el usuario puede cancelar o detener. Audio y fuentes disponibles varían. | Web sin instalación para el caso común. Validar versiones, navegador administrado y permisos corporativos. |
| Android (Chrome/Samsung Internet móviles) | El link funciona para orientación, chat o compartir contenido dentro de la misma página; no asumir captura del dispositivo vía web. | Una pestaña no puede capturar globalmente otras apps. | Una app Android ligera abre `MediaProjectionManager.createScreenCaptureIntent()`. Android muestra consentimiento del sistema; la app transmite la captura con servicio foreground `mediaProjection`. Android 14+ permite al usuario escoger app o pantalla. |
| iPhone (Safari y otros browsers en iOS) | El link puede hacer soporte contextual dentro de la web, pero no capturar de forma general el display y otras apps. | La API de captura web no es una base interoperable en Safari iOS; WebRTC por sí solo solo transporta media que ya se obtuvo. | App iOS pequeña con el selector/captura nativos (ScreenCaptureKit en sistemas compatibles; ReplayKit broadcast en despliegues que lo requieran), picker explícito y transmisión. |

La API web `getDisplayMedia()` está limitada a contextos seguros, necesita activación transitoria del usuario, pide una selección/permisos visibles y entrega un stream que se puede enviar con WebRTC. Los permisos no deben tratarse como una autorización persistente para volver a capturar después. [MDN: `getDisplayMedia()`](https://developer.mozilla.org/en-US/docs/Web/API/MediaDevices/getDisplayMedia) · [W3C Screen Capture](https://www.w3.org/TR/screen-capture/).

Para iOS, la matriz consultada indica que Safari en iOS no implementa `getDisplayMedia()`; Apple sí documenta captura iOS con ScreenCaptureKit y picker del sistema, en un target de app nativa. Hay que confirmar disponibilidad por versión de iOS, ya que la documentación de Apple marca el sample actual para iOS 27+. [Can I use: `getDisplayMedia`](https://caniuse.com/mdn-api_mediadevices_getdisplaymedia) · [Apple: Capturing screen content on iOS](https://developer.apple.com/documentation/screencapturekit/capturing-screen-content-on-ios) · [Apple: ScreenCaptureKit](https://developer.apple.com/documentation/screencapturekit).

En Android la ruta oficial es nativa: `MediaProjection` solicita permiso al usuario para cada sesión. Android 14 añade la elección entre una app y el display completo; versiones recientes muestran controles visibles para detener la captura. [Android: Media projection](https://developer.android.com/media/grow/media-projection) · [Android: App screen sharing](https://developer.android.com/about/versions/14/features/app-screen-sharing).

## Recomendación

Construir una experiencia progresiva:

1. **Desktop, sin instalación:** página de sesión HTTPS con código de un solo uso, explicación del alcance, botón visible para compartir, selector nativo del navegador y un “Detener” permanente. Establecer la llamada con WebRTC; usar TURN cuando la conectividad directa no funcione.
2. **Móvil, cero instalación:** ofrecer guía visual paso a paso dentro de la página (capturas anonimizadas, puntero/instrucciones, compartir una pestaña o documento, chat/voz). Para ayudar en otra app, pedir que vuelva a la llamada si el OS la pausa o limita el navegador.
3. **Móvil, compartir apps externas:** cuando sea imprescindible, ofrecer una app/helper nativa pequeña —Android MediaProjection; iOS picker y captura nativos— enlazada a la misma sesión WebRTC. Explicar por adelantado qué se comparte, limitar duración, no grabar por defecto, indicar al agente cuándo el stream está activo y permitir terminarlo desde la UI y desde el control del sistema.

Así el cliente puede empezar desde un link en cualquier dispositivo; solo una parte de los casos móviles instala el helper. No prometer control remoto ni interacción táctil: la captura de pantalla envía video, no concede control del dispositivo. Separar señalización, autorización de sesión y transporte; usar tokens efímeros y consentimiento renovado por sesión.

## Qué no es posible

- Un link no puede conceder a un agente acceso oculto y continuo a todas las apps del teléfono.
- `getUserMedia()` permite cámara/micrófono con permiso, pero no sustituye una API de captura de pantalla.
- WebRTC es transporte en tiempo real, no una API de captura ni un bypass de permisos del sistema.
- No se puede prometer idéntico comportamiento, selección de audio o continuidad en segundo plano en todas las combinaciones de SO, navegador, políticas empresariales y fabricante. Probar una matriz real de dispositivos antes del lanzamiento.

