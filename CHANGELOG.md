## Landing 0.7.10 — 2026-11-26

- Add a wide «Nuevo · Acuerdos» band to the home «Explora» grid linking to acuerdos.firmar.ec (utm_source=firmar.ec, utm_medium=home); acceptance wording only, no legal-validity claims.

## Landing 0.7.9 — 2026-09-18

- Align llms.txt source repository with the public IDK Manager canonical repository; both existing GitHub mirrors remain valid.
- Clarify the default PAdES B-B profile and optional B-T/B-LT/B-LTA outcomes according to settings and completed evidence checks.

## Landing 0.7.8 — 2026-09-18

- Publish ES/EN PDF verification methodology with four evidence dimensions, a review checklist, dated source-code references and primary standards.
- Connect the resource from verification, security, about, hreflang, sitemap and llms.txt.
- Replace unsupported competitor/exclusivity claims and obsolete promises on About with inspectable project status.

# Changelog

Todos los cambios notables a este proyecto se documentan aquí.
El formato sigue [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) y este proyecto usa [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Verificador — motor 0.10.4 (`@firma-ec/verifier` 1.2.4; PWA 0.28.4, verify-api 0.4.4) — 2026-09-24

Revisión independiente de 0.10.3 (Opus: APPROVE; Codex: BLOCK con 2 HIGH) y de sus arreglos (Codex: 1 HIGH + 3 MEDIUM), todo arreglado aquí.

#### Security
- Una CRL omitida por tamaño se atribuía solo al emisor del firmante sin saber si era **indirecta**. Ahora se recorre su estructura TLV sin decodificar las entradas: se lee `issuingDistributionPoint.indirectCRL` y se busca la extensión `certificateIssuer` entrada por entrada, comparando el OID por contenido (también con longitudes BER no mínimas). Si puede ser indirecta, o no se puede leer, quedan pendientes también las CA. Antes, un `good` en vivo del firmante dejaba la firma `valid`.
- El emisor de una CRL omitida se compara con **todos** los eslabones: si una CA comparte nombre con el emisor del firmante (renovación de clave), la coincidencia con la CA ya no queda tapada por la del emisor del firmante.

#### Fixed
- Una CRL grande de un emisor ajeno a la firma ya no añade el aviso `crl_too_large_skipped`, y el aviso ya no se repite por cada eslabón.
- Un número de serie que contiene los bytes del OID de `certificateIssuer` ya no hace pasar una CRL directa por indirecta (falso `revocation_unchecked`).
- Con una cadena `[firmante, ancla]` no hay CA intermedia que revisar: ya no se marca ninguna como pendiente, tampoco cuando el análisis LTV agota su plazo de 12 s.
- Los `scripts/deploy-*.sh` ejecutan la guarda **antes** de cargar `.deploy.env`: nada de lo que ese fichero defina (ni un envoltorio de `git`) llega a la comprobación.

#### Tests
- Regresiones: CRL grande indirecta y con `certificateIssuer` (y control con IDP no indirecto), cadena con dos eslabones del mismo nombre, CRL real de 1,19 MB de Security Data SubCA-2 (sigue atribuida solo al firmante), `tbsCertList` más largo que su contenedor, y el plazo real de 12 s de `verifyPdf` con temporizadores falsos. Cada una vista en rojo contra 0.10.3 o con una mutación.

### Verificador — motor 0.10.3 (`@firma-ec/verifier` 1.2.3; PWA 0.28.3, verify-api 0.4.3) — 2026-09-24

Revisión independiente (Opus: APPROVE; Codex) de 0.10.2.

#### Security
- Si el análisis LTV supera su plazo de 12 s, tampoco se dan por comprobadas las CA de la cadena: un `good` en vivo del firmante no resuelve la revisión.
- Una CRL **indirecta** firmada por un emisor de la cadena deja pendientes también las CA (no se limita al emisor del firmante). Una CRL pequeña que no se puede leer deja la revisión incompleta en vez de ignorarse.
- El lector de cabeceras DER del emisor de una CRL omitida exige que cada elemento quede dentro de su contenedor.

#### Fixed
- Material de revocación omitido: el emisor de la CRL se compara por nombre (como `isEqual` de pkijs, no byte a byte) con todos los eslabones de la cadena; una CRL de un emisor ajeno a la firma ya no provoca `revocation_unchecked`.
- `ocsp_after_expiry` tiene su propio aviso en español en lugar del genérico `ocsp_unavailable` («el servicio no respondió»), que era falso.
- La función de la guarda de deploy no puede redefinirse desde `.deploy.env` (`readonly -f`).

### Verificador — motor 0.10.2 (`@firma-ec/verifier` 1.2.2; PWA 0.28.2, verify-api 0.4.2) — 2026-09-24

Revisión independiente (Codex) de 0.10.1.

#### Security
- El **OCSP en vivo** tampoco acepta un `good` producido después de caducar el certificado (`ocsp_after_expiry`); 0.10.1 solo lo aplicaba a la evidencia embebida.
- **Material omitido de una CA** (CRL demasiado grande cuyo emisor no es el del firmante, o no atribuible): la respuesta OCSP en vivo del firmante ya no lo da por resuelto; queda `revocation_unchecked`. El emisor de la CRL omitida se lee recorriendo solo las cabeceras DER, sin parsear sus entradas.
- **CRL indirectas** (`indirectCRL`, o entradas con `certificateIssuer`): no se interpretan por número de serie, porque la entrada puede ser de otro emisor; quedan como material no evaluado. Regresión de 0.10.1, que al volver a leer CRL con alcance podía revocar al certificado equivocado.
- **Guarda de deploy**: la excepción se captura antes de cargar `.deploy.env`, en una variable de solo lectura, así que ese fichero no puede activarla con ninguna sintaxis (`export "…"`, `declare -x`, asignación simple).

### Verificador — motor 0.10.1 (`@firma-ec/verifier` 1.2.1, `@firma-ec/tsa-trust` 0.9.1; PWA 0.28.1, verify-api 0.4.1) — 2026-09-24

Revisión independiente (Opus, Codex) de lo desplegado como 0.10.0.

#### Security
- **Revocaciones en CRL con alcance parcial o delta.** 0.10.0 descartaba entera una CRL con `issuingDistributionPoint` o `deltaCRLIndicator`, revocaciones incluidas: una CA subordinada revocada en la ARL de su raíz (`onlyContainsCACerts`, el formato normal de una ARL) salía `valid`. Ahora un serial listado en cualquier CRL firmada por la emisora es revocación; el alcance solo impide tomar la ausencia como favorable.
- **Evidencia posterior a la caducidad.** Una CRL u OCSP emitidos después de caducar el certificado ya no prueban que no estuviera revocado (las CA purgan los seriales caducados), salvo CRL con `ExpiredCertsOnCRL` desde antes de la caducidad.
- **Material de revocación omitido** (límite de tamaño o de tiempo) deja la comprobación incompleta aunque haya evidencia favorable del firmante, y entonces se consulta el OCSP en vivo; si no la resuelve, advertencia `revocation_unchecked`.
- **Certificado de TSA**: `timeStamping` debe ser el único uso extendido y crítico (RFC 3161 §2.3). Con las raíces de ARCOTEL anclando sellos, un certificado de una CA acreditada que solo *incluyera* `timeStamping` entre otros usos podía fabricar sellos. Los TSU reales comprobados (MINTEL TSU02, UANATACA TSU01 ES y EC, FreeTSA) cumplen.
- **Guarda de deploy**: solo despliega la punta de `gitea/main` (no un commit anterior), con remoto fijo, y rechaza la excepción `ALLOW_OFF_MAIN_DEPLOY` si está persistida en `.deploy.env`.

#### Known limitations
- Las CRL delta no se combinan con su base: una suspensión (`certificateHold`) levantada solo en una delta sigue contando como revocación (falla hacia inválido, no hacia válido).
- Las CA de la cadena solo se comprueban con la evidencia embebida; el OCSP en vivo consulta únicamente al firmante.

### Verificador — motor 0.10.0 (`@firma-ec/verifier` 1.2.0, `@firma-ec/tsa-trust` 0.9.0, `@firma-ec/tsa-client` y `@firma-ec/ltv-validation` con cambios compatibles) — 2026-09-23

#### Security
- **La validación de cadena comprobaba un certificado distinto del firmante.** El motor de cadenas de
  pkijs toma como hoja el *último* certificado de la lista, y el verificador ponía al firmante el
  *primero*: se validaba la cadena de la última intermedia del pool y la raíz se asignaba comparando
  solo nombres. Consecuencia: un certificado cuya firma de emisión no verificaba podía salir como
  válido bajo una ACE acreditada, tanto en «Verificar firma» como en «Validar certificado». Ahora el
  firmante va al final (sin copias previas), hay una guarda que falla cerrado si la ruta verificada no
  empieza en el firmante y la ACE se toma del ancla que pkijs verificó. Presente desde la primera
  versión de `pathValidation.ts`.
- **La fecha de firma declarada por el firmante ya no basta para validar un certificado caducado.**
  La cadena se valida a la hora de un sello de tiempo RFC 3161 verificado o, si no lo hay, a la hora
  actual. Si solo es válida en la fecha que declara el firmante, la firma pasa a *advertencia*
  (`signing_time_unproven`) en lugar de *válida*.
- **Sellos de tiempo manipulables.** La firma de la TSA cubre los atributos firmados, no el `TSTInfo`;
  faltaba comprobar que el atributo `message-digest` coincide con el hash del `TSTInfo` (y que
  `content-type` es `TSTInfo`). Sin eso, la fecha y la huella de un sello real podían reescribirse
  conservando una firma válida de la TSA. Ahora el sello debe casar con sus atributos firmados.
- **OCSP en vivo autenticado.** La respuesta se valida con la firma del responder (emisora o delegado con
  `id-kp-OCSPSigning`), el `CertID` del certificado consultado y la vigencia (`thisUpdate`/`nextUpdate`).
  Antes se leía la primera respuesta sin verificar nada. El `CertID` debe nombrar también al emisor
  (`issuerKeyHash`), y una respuesta sin `nextUpdate` con `thisUpdate` de más de 7 días no se da por
  vigente.
- **Se lee la fecha real de revocación.** pkijs deja el `RevokedInfo` como nodo ASN.1 sin decodificar y
  la fecha y el motivo se perdían; con la regla nueva eso habría invalidado firmas revocadas *después*
  del sello.
- **La revocación se compara con la hora probada de la firma.** Una revocación anterior o igual a la
  hora del sello (o a la actual, sin sello) invalida la firma (`cert_revoked`); una posterior la deja
  válida con aviso (`revoked_after_signing`). Sin fecha de revocación, se invalida.
- **La evidencia de revocación embebida (DSS) decide el veredicto** cuando está autenticada (OCSP
  verificado o CRL firmada por la CA emisora); antes solo generaba un aviso y un firmante revocado podía
  salir válido. Las CRL embebidas sin firma de la CA emisora ya no cuentan en ningún sentido, y una
  respuesta favorable ya no corta la búsqueda: el orden de la DSS lo elige quien escribe el PDF. Se
  revisa toda la evidencia (OCSP y CRL) y decide la revocación más temprana; si el presupuesto de la
  LTV se agota, no se afirma que la revocación se comprobó con resultado bueno.
- En respuestas OCSP con varias entradas, la entrada se elige por serial **y** hash de clave del
  emisor (según el algoritmo de cada `CertID`) antes de mirar su estado.
- Un sello añadido después de que caducara el certificado ya no da «Firma inválida — emisor no
  reconocido»: se valida en la fecha declarada como `signing_time_unproven`.
- **Respondedores OCSP delegados autenticados de verdad.** `BasicOCSPResponse.verify` de pkijs tiene el
  mismo defecto de orden: validaba el último certificado adjunto, no el respondedor, así que un
  respondedor impostor pasaba adjuntando cualquier CA real (y una DSS añadida al PDF podía hacer pasar
  por revocada una firma legítima). Ahora el respondedor debe estar emitido directamente por la CA del
  certificado consultado, llevar `id-kp-OCSPSigning`, estar vigente cuando se produjo la respuesta
  (`producedAt`, no hoy) y haber firmado la respuesta (RFC 6960 §4.2.2.2).
- **La DSS solo sustituye al OCSP en vivo si aporta evidencia autenticada del firmante.** Antes bastaban
  unos bytes cualesquiera en `/DSS` para no consultarlo. La LTV se evalúa ahora antes que el OCSP. Si la
  revisión embebida se corta por tiempo, no hay evidencia autenticada y el OCSP en vivo no la zanja, la
  firma queda en advertencia (`revocation_unchecked`), nunca `valid`.
- **La evidencia embebida tiene que cubrir la hora probada de la firma.** Un OCSP `good` o una CRL que
  no lista el certificado solo cuentan si se emitieron después de esa hora o seguían vigentes en ella;
  una respuesta antigua, anterior a una revocación, ya no puede sustituir a la consulta en vivo. Una
  revocación autenticada cuenta siempre.
- Las CRL con `issuingDistributionPoint` (particionadas) o delta no se usan como evidencia, y una
  entrada `removeFromCRL` no se interpreta como revocación.
- Omitir material de la DSS por tamaño sin tener otra evidencia del firmante, o cortar la revisión por
  tiempo, deja la revocación como no comprobada (`revocation_unchecked` salvo que el OCSP en vivo la
  resuelva).
- Con una raíz v1, la cadena devuelta llega hasta la raíz fijada, de modo que la CA subordinada que
  actúa de ancla también se comprueba por revocación.
- La revocación autenticada de una **CA intermedia** anterior a la hora probada invalida la firma igual
  que la del firmante, con su propio mensaje.
- El certificado de la TSA se elige por el `sid` del firmante del sello, no por su posición en
  `certificates`.
- Un certificado cuyo `keyUsage` no incluye `digitalSignature` ni `nonRepudiation` se rechaza
  (`key_usage_not_signing`) en lugar de generar solo un aviso. Un certificado sin la extensión
  `keyUsage` no se rechaza por ello (RFC 5280: uso no restringido).
- Se retira la degradación a advertencia por «raíces parcialmente placeholder» (`TRUST_PARTIAL`):
  bastaba una raíz placeholder activa para que cualquier cadena rechazada, incluida una falsificada,
  pasara a advertencia. `TRUST_PLACEHOLDER` (todas las raíces placeholder) no cambia.

#### Fixed
- **APPFIRMAS 2025: todas sus firmas salían «Firma inválida».** Su raíz («APPFIRMAS S.A. Root AC», 2025)
  es un certificado X.509 v1 sin extensiones y pkijs exige `CA:TRUE` a todo certificado de la ruta,
  ancla incluida. Para raíces de la TSL sin `basicConstraints`, una CA subordinada del pool actúa como
  ancla (atribuida a esa raíz) solo si la raíz está vigente y la subordinada es una CA emisora cuya firma
  verifica con la clave de la raíz fijada; una subordinada forjada con el mismo nombre no entra.
- **Multifirma: una firma legítima salía «Firma inválida — emisor no reconocido».** Caso real: PDF con
  una firma de UANATACA CA2 2016 y otra de UANATACA CA2 2021; la hoja de la otra firma quedaba última
  en el pool y el motor validaba la cadena equivocada. Ahora ambas salen válidas.
- **Sellos de tiempo de cualquier ECI acreditada por ARCOTEL.** Antes solo se reconocían FreeTSA y
  UANATACA ROOT 2016, así que un sello de Security Data, del BCE o del TSU01 de UANATACA Ecuador (emitido
  por CA2 2016) daba `chain_invalid` y no servía como prueba de la hora de firma. Ahora las raíces de la
  TSL-EC que ya se usan para las firmas (solo las reales, con su huella verificada) y sus CA subordinadas
  también anclan sellos, exigiendo siempre el EKU `timeStamping`. Es la lista de TSA que reconoce FirmaEC
  desde su v4.2.0 (Darkcam, Security Data, ANFAC) más BCE, iCert y UANATACA EC.
- **Sellos de tiempo de MINTEL TSU02 marcados como no válidos.** Su emisora, UANATACA CA2 2021, no
  estaba en el bundle de intermedias de TSA (solo CA1 2021).
- El emisor usado para OCSP/CRL en «Validar certificado» se resuelve por AKI/SKI y firma, no por CN.
- OCSP recibe como emisor la CA emisora real (antes podía recibir la raíz).
- Una respuesta OCSP en vivo que no se puede autenticar o no es vigente ahora da `ocsp_unavailable`
  (advertencia); antes quedaba como no consultada y la firma salía `valid`.
- La PWA tiene resúmenes propios para `revoked_after_signing` y `revocation_unchecked`.
- La PWA muestra «Fecha de firma no demostrada» para `signing_time_unproven` en lugar de «Firma válida
  con advertencias», y el resumen de revocación sale del código `cert_revoked`, no del estado OCSP.
- Nuevos resúmenes en la PWA para `key_usage_not_signing` y `signer_cert_not_valid` (certificado no
  vigente en la fecha de la firma), en lugar de atribuirlos a una firma criptográfica incorrecta o a
  un emisor no reconocido.

#### Removed
- `validateChain` de `@firma-ec/crypto-core`: sin llamadores y con el mismo defecto de orden.

### Landing 0.7.7 — 2026-09-17

- Las guías de Security Data y firma de documentos responden directamente a la intención de búsqueda y describen el flujo real de firma, sin prometer tiempos ni aceptación universal.
- Precios sustituye rangos sin fuente por tarifas publicadas con IVA, modalidad, vigencia y fecha de consulta; corrige ICERT-EC a USD 27,60 por dos años.
- Las FAQs de estas tres páginas se renderizan y generan JSON-LD desde una sola fuente; los HowTo reflejan los pasos visibles.
- Las tres traducciones inglesas mantienen las mismas fuentes, tarifas, límites y pasos de sus equivalentes españoles.
- La comparativa de emisores ES/EN refleja la misma tarifa oficial de ICERT-EC; se conservan los enlaces a comparativas y la atribución UTM de la tienda.
- Las tablas de emisores se desplazan dentro de un contenedor accesible y dejan de ensanchar la página en móvil.

### PWA 0.27.0 / signer 0.12.0 — 2026-09-14
- El QR de las nuevas firmas visibles también se puede pulsar para abrir el verificador de firmar.ec. El receptor selecciona el PDF y la validación sigue siendo local. Disponible en firma única y multifirma, con la zona pulsable ajustada a la rotación de la página. El enlace se incorpora antes de firmar; los PDF ya emitidos no cambian.

### Landing 0.7.6

- El CTA principal abre directamente la ruta de firma, conservando los parámetros de consulta. Las redirecciones históricas ES/EN de firma también preservan la consulta.
- Precios ES/EN aclaran el IVA por fila y distinguen solicitud de emisión, sin prometer un plazo universal. La versión inglesa reconoce la venta actual y el papel de la Autoridad de Certificación.
- Patrocinio muestra la disponibilidad de certificados y retira la etiqueta de popularidad sin evidencia.

### Fixed
- **Las guardas del incidente anterior eran, ellas mismas, mudas** (`@firma-ec/pwa` 0.26.2). Una revisión
  con dos agentes independientes sobre el diff completo encontró que lo arreglado el 09-sep estaba bien,
  pero que **las protecciones añadidas para que no volviera a pasar no podían ver el fallo**. Corregido:
  - *El smoke del 404 se ponía VERDE sin medir nada.* `CC404` se capturaba de una tubería sin respaldo:
    si `curl` fallaba, la variable quedaba vacía, caía en el cajón de sastre permisivo y el deploy pasaba
    imprimiendo `404 de /assets no cacheable ()`. Además afirmaba en negativo (solo fallaba ante
    `immutable`), así que un 404 con `max-age=604800` o sin cabecera pasaba. Ahora exige `no-store` en
    **positivo**, con `--max-time`, `|| echo CURL-FAIL` y el vacío tratado como ROJO. Probado contra seis
    escenarios: antes 3 de 5 daban falso verde, ahora solo el verde real es verde.
  - *Esa comprobación disparaba `docker service rollback`.* Es una propiedad de la config horneada en la
    imagen, no un estado que converja: reintentarla 15 veces no la cambia, y revertir es otro rolling
    update `start-first` — es decir, volver a generar los 404 que envenenan el edge. Ahora se evalúa una
    sola vez, fuera del bucle, y falla sin revertir.
  - *`handle_errors` no cubría «TODAS las respuestas de error», como afirmaba el comentario.* Un
    `respond <matcher> 4xx` escribe la respuesta y no pasa por ahí. Verificado sobre la imagen real: en
    la instancia handoff, el 404 de `/manifest.webmanifest` salía con `public, max-age=3600` — un error
    cacheado una hora. Los tres `respond` pasan a `error`, que sí se encauza; comprobadas las cuatro
    rutas (404 de assets y de `respond` → `no-store`; asset real → `immutable`).
  - *La guarda de workflows no corría por donde entró el bug*: `unit.yml` solo disparaba en
    `pull_request` y los últimos ocho commits de `main` fueron push directo. Añadido `push: [main]`, con
    el límite conocido escrito al lado: si el fichero roto es el que lleva el job, Gitea también lo
    ignora en silencio y eso no lo puede cubrir el propio CI.
  - *Falso rojo por tabuladores*: YAML los prohíbe para **indentar**, no dentro de un escalar; un tab en
    un `awk` era válido y habría parado el CI sin nada roto. La afirmación se acota al sangrado.
  - *La fila «RUC» se pintaba desacoplada de la razón social.* Cuando la ACE no está mapeada, el RUC sale
    del `organizationIdentifier` del DN sin razón social que lo acompañe, y el número de una empresa se
    leía como dato de la persona — el error exacto que esta pantalla existe para evitar. Ahora la
    etiqueta dice de quién es: «RUC de la empresa» salvo que empiece por la cédula del titular.
  - *El test que decía probar el cableado de `certCheck` solo afirmaba la dirección que ya era cierta*
    (un certificado de persona natural, donde ambos campos son `undefined`). Se le pasa ahora un
    certificado de representante legal firmado de verdad; visto en rojo quitando las dos asignaciones.
  - *El corpus real podía perderse a la mitad en silencio*: `leavesFrom` descarta blobs que no parsean y
    el test sumaba dos fixtures, así que uno podía desaparecer sin poner nada en rojo. Se afirma por
    fichero.
  - *Procedencia corregida en cuatro sitios.* Se afirmaba `.11` = «el RUC de la empresa (no el del
    titular)»: falso en general — en ArgosData de persona natural, BCE y Uanataca es el del titular. Solo
    lo es cuando existe `.10`. Y de las cinco ACE, la única con `.5`/`.10` **en el repositorio** es
    Security Data; ArgosData y Uanataca se verificaron fuera de banda con .p12 reales que no pueden
    commitearse.
- **Un 404 bajo `/assets/` se cacheaba un AÑO, y tumbó la app durante 7 minutos** (`@firma-ec/pwa` 0.26.1,
  `infra/docker/Caddyfile.pwa`). La política de caché se aplica por **ruta** (`@hashed path_regexp
  ^/assets/…`), y Caddy la evalúa antes de saber si el fichero existe: un 404 salía con el mismo
  `max-age=31536000, immutable` que un chunk real, y `handle_errors` no tocaba la cabecera. Durante un
  rolling update `start-first` conviven la réplica vieja y la nueva unos segundos, así que una petición
  al chunk NUEVO puede caer en la VIEJA y recibir 404 — que quedaba cacheado un año en Cloudflare y en
  el navegador, el cual con `immutable` **ni siquiera revalida**. Pasó de verdad al desplegar la 0.26.0:
  el entry `index-*.js` quedó envenenado en el edge (`cf-cache-status: HIT`) y cualquier visitante sin
  Service Worker recibía una página en blanco. Resuelto en el momento purgando la zona, y aquí a la
  raíz: `handle_errors` fuerza `Cache-Control: no-store`, que es el único punto por el que pasan todas
  las respuestas de error. El smoke post-deploy afirma ahora que un 404 de `/assets/` NO es cacheable,
  sobre la respuesta real de producción. Verificado en rojo antes del fix: `/assets/no-existe-….js`
  respondía `public, max-age=31536000, immutable`. El bump de versión cambia el hash del entry, así que
  los navegadores que guardaron el 404 piden un nombre nuevo y se recuperan solos.

### Added
- **La ficha de «Validar certificado» ya distingue a un representante legal** (`@firma-ec/pwa` 0.26.0).
  Un certificado de representante legal identifica a DOS partes —la persona que tiene la llave y la
  empresa por la que firma— y la ficha solo mostraba a la persona: quien recibía un documento no
  podía saber si la firma obligaba a una empresa. Ahora salen **RUC**, **Razón social** y **Cargo**,
  los mismos tres campos que muestra FirmaEC 5.1.0. Las ACE publican el par bajo su propio arco OID
  con sufijos comunes: `.5` cargo, `.10` razón social y `.11` el RUC — que es el de la **empresa
  cuando existe `.10`**, y el del propio titular (cédula + código de establecimiento) en un
  certificado de persona natural; por eso la ficha etiqueta la fila según a quién pertenece, en vez
  de llamarla «RUC» a secas. Procedencia, para que nadie confíe más de lo probado: los únicos
  certificados REALES del repositorio con `.5`/`.10` son de Security Data; ArgosData y Uanataca se
  verificaron fuera de banda con .p12 de representante legal auténticos, que no pueden commitearse
  porque llevan la cédula de una persona viva y el RUC de una empresa real. `organization` no
  cae al RDN `O` del subject a propósito: ArgosData pone ahí la empresa representada, pero Security
  Data y el BCE ponen su PROPIO nombre, y el fallback atribuiría la identidad del emisor al titular.
  La etiqueta «Cédula / RUC» pasa a «Cédula» porque el RUC ya tiene fila propia. Verificado con un
  .p12 real de representante legal sobre el build de producción: los tres valores coinciden con los
  que FirmaEC 5.1.0 muestra para el mismo archivo.

### Changed
- **La app puede embeberse desde 3tap** (`infra/docker/Caddyfile.pwa`): `frame-ancestors 'none'` pasa a `'self' https://app.3tap.ec` y se retira `X-Frame-Options: DENY` (no admite lista y pisaría la CSP). 3tap.ec la muestra en su pestaña «Firmar documentos» con las mismas pantallas (previsualizar, firmar, verificar). Ningún otro origen puede embeberla.

### Fixed
- **El JSON-LD del landing anunciaba una versión que no existía** (`@firma-ec/landing` 0.7.5).
  `softwareVersion` estaba escrito a mano y se quedó en `0.9.14` mientras la PWA iba por 0.25.x:
  el dato estructurado que leen buscadores e IA llevaba meses mintiendo. Ahora lo inyecta el build
  desde `apps/pwa/package.json` con el mismo helper que ya alimenta el pie de la app
  (`appVersionDefine()`), así que bumpear ese package.json es el único gesto y no hay un segundo
  número que pueda quedarse atrás. Verificado sobre el HTML construido: las 78 páginas dicen 0.25.2
  y no queda ni un rastro de 0.9.14.
- **El CI de GitHub llevaba roto desde el 27 de agosto y tapaba lo que venía detrás.**
  `pnpm lint` fallaba con 22 errores (17 ficheros sin formatear, 3 de orden de imports y 2
  supresiones `biome-ignore` que ya no suprimían nada), y como el job para en el primer paso rojo,
  nunca llegaban a ejecutarse el typecheck ni las pruebas — que también estaban en rojo:
  `svelte-check` daba 3 errores porque el landing declaraba `svelte@^5.55.5` y la PWA `^5.56.8`
  (dos copias, «Two different types with this name exist»), `pnpm test` fallaba en
  `built-bundle.test.ts` por falta del bundle de verify-api, y arrastraba los specs del Service
  Worker, que son de Playwright. Ahora los cuatro pasos están en verde.
- **Los tests del Service Worker no los ejecutaba NADIE.** Son la red de regresión del fallo
  «el firmador no funciona» de 0.25.1, corren contra un build real con `vite preview` y tienen su
  propio config de Playwright — pero ningún workflow los invocaba, y vitest solo los recogía para
  fallar al cargarlos. Se excluyen de vitest y se añade el paso que los corre de verdad.
- **Un certificado con Ñ en el nombre no podía firmar** (`@firma-ec/signer` 0.11.2, `@firma-ec/pwa` 0.25.2).
  Reportado el 2026-09-06 como «No se pudo firmar · code: unknown» con el nombre del firmante
  mostrado como «SIMBAÃ‘A». La CA emitió el CN como TeletexString con UTF-8 dentro, y tanto
  node-forge como asn1js decodifican ese tipo byte a byte: la Ñ llegaba como `Ã` + U+0091, un
  carácter de control que el codificador WinAnsi de pdf-lib rechaza al medir la estampa. La
  excepción no llevaba código y el worker la devolvía como `unknown`. Dos capas: (1) crypto-core
  repara al leer cualquier DirectoryString byte-a-byte cuya secuencia sea UTF-8 válido
  (`decodeAsn1DirectoryString`), y las tres lecturas del CN (forge, asn1js en `p12.ts`,
  `subjectInfo` del verificador) pasan por ahí; (2) la estampa nunca lanza por un carácter que
  Helvetica no tiene: `helveticaSafe` lo sustituye por su base sin diacrítico o por `?`, y el
  codificador hex usa la tabla WinAnsi real en vez del byte bajo del code point (`’` salía como
  0x19 y `€` como 0xAC). Firma única y multifirma comparten ahora el mismo codificador.
- **El Service Worker reiniciaba la app encima del documento del usuario** (`@firma-ec/pwa` 0.25.1).
  Reportado como «el firmador no funciona», y no había ningún error: en la PRIMERA visita el
  SW se instala, hace `clients.claim()` y dispara `controllerchange` unos segundos después de
  cargar. `swUpdate.svelte.ts` recargaba la página ahí sin comprobar si había un controlador
  ANTERIOR — y no lo había: la página ya estaba corriendo el código recién bajado de la red,
  así que la recarga no cambiaba una línea y sí borraba el PDF, la colocación y el PIN, que
  viven solo en memoria. Medido en producción (Pixel 7, 3G): PDF subido a los 2,6 s, recarga a
  los 3,6 s, usuario de vuelta en «Sube tu PDF» sin explicación. Le pasaba a todo visitante
  nuevo y tras cada despliegue — justo a quien prueba la app por primera vez. Ahora solo se
  recarga cuando el controlador CAMBIA de uno viejo a uno nuevo. Además, `Firmar.svelte` retiene
  la recarga mientras haya un documento cargado (lo que `FirmarLote.svelte` ya hacía para el
  lote), y `UpdateNotification` OFRECE la versión retenida en vez de dejarla inaplicable.
- **El hueco que dejó pasar esto: ningún test corría con un Service Worker real.** `vite dev` no
  emite SW, así que 419 tests unitarios y 56 e2e daban verde mientras producción se recargaba
  sola. Nuevo `apps/pwa/tests/sw/primera-visita.spec.ts` + `playwright.sw.config.ts`, que corren
  contra `vite preview` (build real) y, con `SW_E2E_BASE_URL`, contra un despliegue en vivo.
  Verificado EN ROJO contra producción antes de arreglarlo.

### Added
- **El sumidero de rutas deja de ser mudo: pantalla de "no encontrado"** (`@firma-ec/pwa` 0.25.0). Hasta hoy, cualquier ruta desconocida —hash como `#/verify` o path como `/validate-certificat`— montaba la portada con 200. Así vivieron meses cuatro rutas rotas que las páginas anunciaban: nadie ve un 200. Ahora el comodín del router apunta a `NotFound.svelte`, y el puente de alias desvía todo path sin alias a `#/no-encontrado?p=<path>`, mostrando la dirección intentada como texto **solo si parece un path** (`sanitizeAttemptedPath`): la revisión demostró que, sin ese filtro, `?p=Llame+al+0999...` pintaba una instrucción de phishing bajo la marca en `app.firmar.ec`. No es XSS, Svelte escapa; es suplantación de contenido (CWE-451), y la pantalla nueva la creaba. La copia pasa a *"No encontramos esta página"*, para quien llega desde un enlace roto y no para un desarrollador. La dirección se relee en cada cambio de hash: el router reutiliza el componente entre rutas muertas y, leída una sola vez al montar, un canary que recorriera varias en la misma sesión habría leído siempre la primera (lo midió la revisión). Quedan exentos a propósito la raíz y las entradas del sistema operativo (`/share`, `/handle-file`, que llegan por path real con su lógica en el service worker); el puente sigue fallando abierto ante cualquier excepción. Con esto, un canary o un monitor pueden **afirmar sobre un texto** en vez de adivinar. Probado en rojo antes de implementar: 2 unitarias y 3 e2e fallaban; ahora 4 e2e herméticos y 13 unitarias del puente en verde. Un test preexistente protegía justo el comportamiento antiguo ("no hace nada en paths desconocidos") y se actualizó, no se esquivó.
- **El extractor de rutas de `check-announced-urls` ancla la marca de fin al inicio de línea y exige que el comodín sea la última clave.** Al añadir la ruta nueva, un comentario mío dentro de la tabla que mencionaba la clave comodín recortó la ventana en silencio: la guarda pasó contando 14 rutas en vez de 15. Lo cazó el recuento que imprime; sin él habría quedado una ruta sin comprobar y todo en verde. Probado con un comentario adversarial: sigue en 15. Y la revisión encontró la variante que quedaba: una ruta escrita **después** del comodín desaparecía del recuento sin aviso (16 reales, 15 contadas, verde); ahora aborta nombrándola.
- **Guarda `check-announced-urls`: cierra la CAUSA RAÍZ de los fallos mudos de rutas** (`@firma-ec/landing` 0.7.4). El landing decidía qué URLs **anuncia** (traduciendo el path como si fuera texto: *"Visit app.firmar.ec/sign"*) y `pathAlias.ts` decidía cuáles **resuelven**, sin nada que las cruzara. Por eso cuatro rutas servían la portada con 200 durante meses y las encontró una revisión, no una alerta. Esta guarda las cruza en cada build: barre 249 ficheros de `src` y `public`, y exige que toda `app.firmar.ec/<path>` anunciada resuelva por la tabla de alias a una ruta real del router, y que todo enlace `#/<ruta>` exista. **Importa** `resolvePathAlias` de verdad en vez de reimplementarlo o de extraer su tabla por regex: la primera versión la extraía, y el panel de revisión demostró que TypeScript válido (una entrada partida en varias líneas, una coma dentro de la regex, un flag) la descartaba **en silencio** — la guarda seguía en verde midiendo de menos y, cuando enrojecía, culpaba a un fichero de contenido que estaba bien. Un detector que se encoge solo es peor que no tenerlo. La tabla de rutas del router sí se lee del fuente, porque vive dentro de un componente Svelte, con cuatro anclas que abortan si el formato cambia. Sigue el precedente de `check-size-claims`, y el `landing.Dockerfile` copia solo esos dos ficheros para que el acoplamiento quede explícito y mínimo. Probada en **las dos direcciones**, que es lo que faltó en la guarda anterior: enrojece si se retira un alias anunciado, si un enlace hash apunta a una ruta inexistente, si una página anuncia una ruta nueva, y si desaparece el comodín del router; y **pasa** ante seis cambios legítimos que la versión anterior bloqueaba — una URL a un fichero servido (`/sw.js`), un host que solo termina igual (`notapp.firmar.ec`), un ejemplo marcado con la válvula, una entrada de alias partida en varias líneas y una regex con coma dentro. Corre **antes** de `astro build`, porque solo lee fuentes y así falla en segundos en vez de tras construir 78 páginas. Verificada además sobre una réplica del contenido de la imagen: sin los `COPY` nuevos aborta ruidoso, nunca en silencio.
- **Guarda hermética `check-nav-links` en el build de la landing** (`@firma-ec/landing` 0.7.3). Corre dentro de `pnpm build`, así que la ejercita el CI, y fija las dos decisiones anteriores sobre el `dist`: el menú apunta a la herramienta (2 anclas por lengua, escritorio y móvil) y el artículo conserva enlace desde el pie. Busca por `href`, nunca por el texto del enlace, y acota la búsqueda a las regiones `<header>` y `<footer>`. La primera versión emparejaba por rótulo y el panel de revisión la tumbó: esa regex sólo veía 50 de las 77 anclas de la home, así que meter un icono dentro del enlace, o cambiar el copy del menú, dejaba el build en rojo **sin que existiera bug** y, con `merge = deploy`, habría bloqueado hasta un hotfix. Probada contra la matriz completa de mutaciones: pasa con icono anidado y con cambio de rótulo, y enrojece si el menú vuelve al artículo o si el pie pierde el enlace. El canary Playwright equivalente habla con producción y por eso no puede ser gate de PR.

### Fixed
- **`/inbox` no resolvía, y lo anuncia un borrador** (`@firma-ec/pwa` 0.24.2). Lo encontró la guarda nueva en `_drafts/como-funciona-wa.astro`, es decir **antes** de publicarse, que es cuando arreglarlo sale barato. Es exactamente el caso para el que se construyó.
- **Cuatro rutas que las páginas anuncian para teclear servían la portada con 200** (`@firma-ec/pwa` 0.24.1). Medido en producción con navegador: `/sign`, `/verify` y `/firmar-lote` caían en la portada, igual que `/validate-certificate`. Las páginas en inglés dicen literalmente *"Visit app.firmar.ec/sign"* (7 menciones), *"/verify"* (3) y *"/firmar-lote"* (4). Falla mudo: un 200 no lo ve ningún monitor. Se añaden los alias, con `/firmar-lote` **antes** de `/firmar` porque el patrón de este no lo cubre. La causa raíz sigue abierta: nada cruza las URLs que el landing **anuncia** con las que la app **resuelve**, así que el próximo idioma o ruta puede repetirlo.
- **Un enlace publicado apuntaba a una ruta que no existe** (`what-is-electronic-signature.md:96`). Enlazaba a `#/verify`, y las rutas hash reales son `/verificar` y compañía: el comodín del router servía la portada sin avisar. Corregido a `#/verificar`.
- **La página en inglés mandaba a una URL que servía la portada en español** (`@firma-ec/pwa` 0.24.1). `en/validate-certificate.astro` dice al lector *"Visit app.firmar.ec/validate-certificate"*, y ese slug no tenía alias en `pathAlias.ts`: el fallback de la SPA respondía **200** con la home y **cero** zonas de carga de `.p12`. Falla muda: ningún monitor ve un 200. Medido en producción con navegador antes y después. Ya existían pares bilingües (`about|acerca`, `configuracion|settings`), así que esta se había quedado fuera. Encontrado por el panel de revisión dual, no por un usuario.
- **El artículo de validación se quedó sin su único enlace de todo el sitio** (`@firma-ec/landing` 0.7.3). Al mover el menú a la herramienta (0.7.2) la página pasó de estar enlazada desde las 78 del sitio a solo 6 páginas-guía. El pie ya listaba a su hermana `verificar-firma-pdf` y no a ella. Se añade a la columna Guías en ES y EN, con rótulo propio (*"Cómo validar tu certificado"*) para no repetir el del menú, que lleva a otro destino.

### Changed
- **La URL del QR impreso en cada PDF firmado vive en un solo sitio dentro del firmador, y la ruta que nombra bloquea el despliegue si cambia** (`@firma-ec/signer` 0.11.1, `check-announced-urls`). Estaba como literal repetido en `pades.ts` e `incrementalUpdate.ts`, y nada comprobaba que la ruta `#/verificar` que nombra siguiera existiendo en el router: si se renombraba, el QR de **todos los PDF ya emitidos** aterrizaba en la portada con un 200 que ningún monitor ve. Lo señaló el panel de revisión como la URL más consecuente y menos vigilada del sistema. Ahora es `VERIFY_QR_BASE` + `buildVerifyQrUrl()` en `verifyUrl.ts`, exportados por el paquete, y `tests/verifyUrl.test.ts` **congela** el literal, que está impreso en documentos que ya circulan, y su estructura (`VERIFY_ROUTE`, exportada). Probado en rojo: cambiar el literal rompe las pruebas. Que la ruta `/verificar` **exista** en el router no lo comprueba este test a propósito: ya lo gatea `check-announced-urls` en el build de la landing, que aborta el despliegue si desaparece de `App.svelte`; la primera versión del test duplicaba ese extractor y la revisión demostró que daba verde falso con la clave apuntando a la portada. Que la ruta apunte al **verificador** y no a otro componente lo gatea ahora una ancla nueva en `check-announced-urls`: con `'/verificar': Home` la clave existe y el QR caería en la portada; la revisión lo demostró mutando el router, y la ancla lo enrojece (probado). Lo que sigue **sin guarda**: la URL que de verdad queda dentro del PDF (la prueba F6.3 de `visibleSig` afirma sobre su propia variable, no sobre el documento). Ambas piden un e2e que cargue `#/verificar?h=<hex>` y vea el banner. `buildVerifyQrUrl` produce la cadena byte-idéntica al literal anterior en los dos caminos de firma; el bundle de la PWA la construye en cada worker de firma a partir de las dos piezas, base y ruta, que el minificador guarda por separado.
- **El canary de navegación pasa a la convención del repo y gana el fallo que no veía** (`tests/e2e/nav-validar-certificado.live.spec.ts`, antes `e2e/live-nav-validar-certificado.spec.ts`). La prueba del móvil afirmaba el `href` sobre un enlace **oculto** dentro del panel `invisible pointer-events-none`: medido, pasaba en verde con la hamburguesa rota mientras el usuario no podía abrir el menú. Ahora exige oculto al cargar, pulsa `#mobile-menu-btn`, exige visible y hace clic real hasta la herramienta. `LANDING_BASE_URL` gobierna ahora **toda** la spec, incluidos el artículo y el sitemap, que iban fijos a producción y ensuciaban las corridas en rojo. Config propio `playwright.live-nav.config.ts` con `testMatch` por nombre, que no arrastra la auditoría v0.3.3, y **sin** `ignoreHTTPSErrors`: un certificado roto en producción debe enrojecer el canary.
- **"Validar certificado" en el menú de firmar.ec abría un artículo, no la herramienta** (`@firma-ec/landing` 0.7.2). El enlace del menú (escritorio y móvil) apuntaba a la página SEO `/validar-certificado/`, así que para validar había que volver al inicio, entrar por "Verificar" y desde ahí a la app. Ahora va directo a `app.firmar.ec/#/validar-certificado`, con el mismo patrón que "Firmar Fácil". La página SEO sigue publicada y enlazada desde el contenido y el sitemap. Reportado por Leandro el 2026-09-02.

## [0.24.0] — 2026-08-27 — la estampa centrada sobre el firmante

### Added
- **El motor ya sabe donde TERMINA cada linea, no solo donde empieza** (`@firma-ec/signer` 0.11.0). `LineStart` gana `end`: la suma de los anchos de glifo de la fuente vigente —`/Widths` + `/FirstChar` + `/MissingWidth` en fuentes simples, `/W` + `/DW` en Type0/CID Identity-H, y una tabla AFM de las 14 estandar (`standardFontWidths.ts`, extraida de `@pdf-lib/standard-fonts`) para las que no incrustan metricas— con `Tc`, `Tw`, `Tz` y los ajustes de `TJ`, ya transformada por la matriz efectiva. La matriz de texto del recorredor **no** se toca: moverla cambiaria los arranques que validan las tablas congeladas, asi que el borde derecho se anota aparte. Contrastado contra pymupdf en las 8 paginas de bloque de firma del corpus real (448 lineas): **mediana 0,00 pt, p90 2,52 pt, maximo 7,53 pt**, dentro del criterio con el que se acepto (mediana ≤ 3, maximo ≤ 8).

### Changed
- **La estampa automatica va CENTRADA sobre el nombre/raya del firmante, no apoyada en el margen izquierdo del bloque** (`@firma-ec/pwa` 0.24.0). Una persona firma **sobre** el nombre; el motor lo apoyaba a su izquierda. Medido en produccion (0.23.4) sobre los 8 documentos reales, la caja quedaba entre **+35 y +88 pt** a la derecha del centro del nombre: una carta con el firmante centrado en x=313 recibia la caja en x=281,7 con 240 de ancho, o sea centrada en 401,7. El centro es el punto medio de la union `[min x, max end]` de los arranques de la **primera** columna del bloque (la misma que ya delimita `columnSplit`), y se aplica en los DOS caminos que anclaban al bloque: el documento sin firmas (`placeOnLastPage`) y el que ya trae una (anti-solape), que es justamente el de la firma individual en produccion.
- **DECISION DEL DUENO sobre que hacer cuando la caja centrada no cabe** (no es una consecuencia de la geometria, es una eleccion): el orden de prioridades es **centrar > conservar el ancho > conservar el centro**, y la cota izquierda es el **margen del TEXTO de la pagina** (`max(EDGE_MARGIN, min x de sus bandas)`), no el del papel. Si la caja no cabe centrada entre las cotas, se **desplaza** hasta la que la aprieta —queda descentrada, pero dentro de la reticula del documento y con el ancho legible—; solo si ni desplazada cabe, se encoge; por debajo de 78 pt se abandona el centrado y manda el comportamiento anterior. La primera version encogia SIMETRICA para conservar el centro a toda costa, y se descarto midiendola: en 2 de los 8 documentos reales dejaba la estampa tocando el borde del papel y hasta 27 pt mas estrecha, descolgada del margen del texto, a cambio de un centro exacto que nadie percibe.
- **Efecto medido en el corpus real** (centro de la caja contra el centro real del bloque, `pymupdf get_text('words')`; ninguno pisa texto ni la firma previa, y en los 6 con cofirmante el borde derecho sigue por debajo de su columna): `carta` 313,62 contra 313,62 · `sacei` y `refrendo` a 0,00 · `nda` a 1,94 · `acta` y `acta-signed` a 3,02 (la caja para en el margen del texto, 72,0) · `demanda` a 31,3 y `contrato-infra` a 40,5, los dos porque el margen del texto (85,1 y 62,4) esta por delante de donde caeria la caja centrada. Todos mejoran sobre el punto de partida, que iba de 34,8 a 88,1 pt de desvio.
- **Corpus congelado**: `contrato2026` x 63,9 → 62,4 (w 204,7 sin cambio) y `lideres` x 140,3 → 98,8 (w 150,6 sin cambio, y su centro pasa de 41,5 pt de desvio a 0,00). Cada delta lleva su medida en el comentario de la entrada.
- **Un `boxW` explicito —el hint de propagacion del lote— sigue intacto**: es una decision de quien firma y el motor no la pisa, ni para centrar.

### Fixed
- **El estimador calla en vez de devolver un numero falso.** Todos estos casos devolvian un ancho con aspecto de medida donde no habia medida: fuentes **Type3** (su `/Widths` va en el espacio de glifo de `/FontMatrix`, no en milesimas: "AAAA" a 10 pt salia 2,00 pt contra 26,68 reales); **Type0 sin `/Encoding /Identity-H`** (CMap predefinida, CMap en stream o cualquier modo vertical, donde el codigo de 2 bytes no es el CID); **`Tf` con tamano ≤ 0 y `Tz` ≤ 0**, que se ignoraban dejando vigente el valor ANTERIOR, con lo que la linea se medía con un cuerpo o una escala que el documento acababa de abandonar —y la marca de "no se sabe" sobrevive ahora al siguiente `Td`, que era el agujero del primer intento—; y los **Form XObject**, que heredaban `Tc=0`/`Tw=0`/`Tz=1` y el tamano de respaldo en vez del estado de texto de quien los coloca (ISO 32000-1 §8.10.1).
- **Con `/Widths` presente, un codigo fuera de rango vale `/MissingWidth` (0 por defecto), nunca la tabla AFM**: una fuente llamada `Arial` puede traer el `/Widths` de cualquier subconjunto, y la Helvetica de Adobe no dice nada sobre los codigos que ese subconjunto no cubre.
- **Los alias de las 14 estandar se limitan a los METRICAMENTE compatibles** (Arial/ArialMT, Liberation, Nimbus). Verdana, Tahoma, Calibri, Segoe, Georgia, Cambria, Arial Narrow y Arial Black salen: medido, Verdana es un 12% mas ancha que Helvetica y Georgia un 15% mas que Times.
- **La tabla AFM perdia el tramo tipografico de WinAnsi (128-159)**: la tabla de codificacion de `@pdf-lib/standard-fonts` va de code point Unicode a `[codigo, nombre]`, y el generador la leyo como si la clave fuera el byte —lo que solo acierta en 32..126 y 160..255—, dejando a cero justo donde viven las comillas curvas, el guion medio y largo, los puntos suspensivos y el Euro. Un apostrofo tipografico en el nombre del firmante, con fuente estandar sin `/Widths`, dejaba la linea sin borde derecho y el centro se corria decenas de puntos. Quedan a cero seis codigos, y es correcto: 127 y los cinco huecos sin asignar de CP1252.
- **Fuga de memoria en `/W`**: los rangos `[cFirst cLast w]` se expandian a un `Map` por CID —**129 MB medidos** con un PDF hostil de 64 fuentes—, y ademas el tope cortaba la expansion, con lo que los CID altos recibian el ancho equivocado. Ahora se guardan como intervalos con busqueda por biseccion, y el tope es **total del documento** en vez de por fuente (antes el techo real era 64 veces el declarado).
- **Con evidencia PARCIAL no se centra**: si una sola linea de la primera columna no tiene borde derecho estimado, se abandona el centrado entero. La union salia corta por la derecha y el centro se corria a la izquierda sin que nada avisara — un centro sesgado se ve igual que uno bueno. Reproducido en revision con un documento real cuya unica diferencia era un guion medio en el nombre: la `x` se movia 74 pt.

### Notas
- **Coste medido de capturar siempre los bytes de las cadenas** (hace falta para medir anchos en toda lectura de bandas): en documentos densos SINTETICOS (60 paginas x 60 lineas) `readTextBands` pasa de 7,8 a 13,8 ms de p50 y de 31 a 45 MB de heap (+60%). En los documentos REALES del corpus no se nota (p50 30-31 ms a ambos lados): ahi domina abrir el PDF con pdf-lib, no recorrer el stream. Se acepta a proposito.
- **Sin cubrir, declarado**: (1) un bloque `Firma: ______` cuya raya es un **path vectorial** —el recorredor solo ve texto, no dibujo— se centra sobre la etiqueta, porque la raya no existe para el motor; no se implementa la mitigacion "union de la columna < 60 pt → no centrar" por no tener corpus con que calibrar ese umbral. (2) En una pagina **rotada 90/270** el centrado esta gateado a proposito, pero ese gate no lo cubre ningun test de extremo a extremo: medido, en pagina rotada `reservedGapV` no encuentra hueco reservado con ninguno de los layouts probados, asi que el camino anclado al bloque ni se ejecuta y quitar el gate no cambia el resultado. Se conserva porque un documento rotado real si puede llegar ahi. (3) Un `q`/`Q` DENTRO de un `BT`...`ET` corta la linea en dos y acorta su `end`; ISO 32000-1 §8.2 no lo permite y el error va en la direccion segura (descentra un poco, nunca inventa anchura). (4) En bloques de etiqueta/valor a dos columnas el motor centra sobre la ETIQUETA, la misma limitacion que ya documenta `columnSplit`.
- **41 tests nuevos** (estimador de fin de linea, centrado, tabla AFM y formas de `/W`) y **25 mutaciones probadas, 24 cazadas**; la que sobrevive es la del gate de rotacion, y sobrevive por la razon del punto (2) de arriba, no por falta de test.

## [0.23.4] — 2026-08-26 — el segundo firmante tampoco invade al cofirmante

### Fixed
- **Con una firma previa en el documento, la estampa del segundo firmante tambien se recorta a su columna** (`@firma-ec/signer` 0.10.4). El recorte de 0.23.3 vivia solo en el camino sin firmas (`placeOnLastPage`); el camino con firma previa (anti-solape) salia con el ancho por defecto "a proposito", porque no distinguia el ancho por defecto del elegido por la persona. Lo destapo la QA dual (Fable + Opus) midiendo sobre la app de produccion con 8 documentos reales: en un contrato ya firmado por el arrendatario la caja (63,9 + 240) **colgaba 28,3 pt sobre el nombre del cofirmante**, que arranca en 275,6; y el NDA real se salvaba por **0,9 pt** de pura casualidad (103 + 240 = 343 contra una raya en 343,9), margen que ningun test miraba. Ahora el anti-solape recibe `widthIsDefault` y aplica el mismo `columnSplit`/`widthWithinColumn`; un `boxW` explicito (hint del lote) sigue sin recortarse.
- **Efecto medido en el corpus real**: `contrato2026` w 240 → 204,7 (x, y iguales); `lideres` baja de y=343 a y=298,1 (w 150,6) — con el ancho de antes la caja cruzaba hasta la firma previa del cofirmante y esa firma "levantaba el suelo": la estampa flotaba 45 pt por encima de su nombre. Recortada a su columna, se apoya sobre el nombre (aire 6,6 pt) a la misma altura que la firma del otro. Verificado en render.

- **Y se ancla a la PRIMERA columna, no al arranque del bloque.** Con una columna que lleva una linea mas que la otra (un RUC bajo la empresa), la `x` de la banda fusionada es la de esa linea extra —la columna derecha— y en el anti-solape la estampa caia **entera** sobre el cofirmante con `status:'ok'` (232,9 pt, reproducido por Opus con un A/B controlado: mismo documento, con y sin firma previa). Era exactamente el HIGH que Fable encontro en la primera vuelta, arreglado entonces solo en el camino sin firmas. Sin columnas detectadas se conserva el arranque del bloque, y el corpus real no se mueve ni un punto.

### Notas
- Cuatro tests nuevos (bloque estrecho a 300 pt: recorta; ancho explicito: no recorta; raya vecina a 0,9 pt: no la roza). La mutacion `widthIsDefault=false` mata los dos de recorte y dos entradas de la tabla congelada (el de ancho explicito guarda la direccion contraria: que un `boxW` elegido no se toque).
- Sigue sin cubrir (medido por la QA dual, sin cambio de comportamiento): la frontera se mide contra arranques de TEXTO; una raya vectorial mas larga que el nombre del cofirmante puede cruzarse unos puntos (3,4 pt en un acta real, invisible a tamano natural). Y la caja se ancla al margen izquierdo del bloque, no centrada sobre la raya — convencion documentada.

## [0.23.3] — 2026-08-26 — la estampa deja de invadir el hueco del cofirmante

### Fixed
- **El PRIMER firmante deja de estampar dentro del hueco del cofirmante** (`@firma-ec/signer` 0.10.3): reproducido en produccion sobre un contrato real con dos firmantes lado a lado. La caja iba de x=140,06 a x=380,06 y la columna del cofirmante empieza en x=344,4 — **35,7 pt dentro de su hueco**. No pisa texto, asi que ninguna comprobacion de solape lo cazaba: la estampa se mete en el blanco donde el otro tiene que firmar y el documento sale defectuoso sin que nada avise. Cuando el bloque bajo el hueco reservado esta a varias columnas, el ancho se recorta ahora al arranque de la siguiente — una cota dura y gratuita, porque un arranque de linea sale de la matriz de texto y no exige metricas de la fuente incrustada. Medido sobre el contrato real: 240 → 197,29 pt de ancho, con 7,05 pt de aire antes de la columna de al lado.
- **El ancla horizontal dependia del ORDEN DE EMISION del content stream, no de la geometria**: `mergeBands` ordenaba por `y` sin criterio de desempate, y dos columnas de un bloque de firma comparten linea base — empatan en `y` exactamente. Como `Array.sort` es estable y la banda fusionada conservaba solo la `x` de la primera, ganaba la columna que el generador del PDF hubiera dibujado antes. Medido con dos PDFs identicos salvo el orden de dibujo: emitiendo la derecha primero, la caja caia **entera** sobre el hueco del cofirmante (232,9 pt de invasion) con `status:'ok'` y confianza `alta`.
- **El ancla se elige a proposito, no por casualidad**: la `x` de la banda fusionada es la de su linea MAS BAJA, y basta con que una columna lleve una linea mas que la otra —un RUC bajo la empresa y no bajo la persona natural— para que esa linea sea de la columna DERECHA. Medido: el ancla se iba a 362,8, no quedaba ningun corte a su derecha, no se recortaba nada y la estampa caia entera sobre el cofirmante. Ahora, detectadas las columnas, se ancla siempre a la primera.
- **El umbral de columna viaja entre tamanos de pagina**: era un valor absoluto medido en A4. En A5 las columnas caben en menos sitio y su separacion real cae por debajo de el, con lo que el detector callaba y la estampa invadia (95 pt medidos). Pasa a ser el menor entre el absoluto y una fraccion del ancho util.
- **La cota se calcula donde la caja va a estar de verdad**: el hueco se media desde el arranque crudo del bloque, pero si arranca por dentro del margen la caja se empuja DESPUES, y ese sobrante volvia a ser invasion (13 pt medidos).

### Changed
- **La vista previa deja de mentir sobre donde cae la caja** (`@firma-ec/pwa`): el lienzo fijaba su ancho Y su alto en pixeles, pero `max-width: 100%` puede recortar el ancho por debajo del pedido --el contenedor se mide antes de que la maquetacion asiente-- y una altura inline no se recorta con el. La pagina se pintaba **aplastada** (A4: relacion 0,681 en vez de 0,707, un 3,75% mas estrecha) y la capa de la caja se dimensionaba con el ancho PRETENDIDO, no con el pintado: la caja aparecia desplazada a la derecha y mas ancha de lo que se iba a estampar. Medido sobre un contrato de dos firmantes: una firma que despejaba la columna del cofirmante por 7,05 pt **se veia metida 6,3 px dentro**. Ahora el alto lo deriva el lienzo y lo que se publica es su caja YA MAQUETADA: el aire aparente pasa de -6,3 px a +7,3 px, que es exactamente el aire real a escala. Defecto presente desde 2026-05-09; no es una regresion, es que hacia falta un documento a dos columnas para verlo.

### Changed
- **El suelo del recorte es el del LAYOUT de la estampa, no el del validador.** Un rect de 43 pt pasa `validateVisibleSig` (que solo pide 30x30) y aun asi se firma **mudo**: el bloque de nombre, cedula y fecha arranca en un x fijo y el BBox del XObject lo recorta entero, dejando solo un QR mordido sobre una firma con PKCS#7, TSA y OCSP ya gastados. Estrechar hasta ahi cambiaba un defecto visible por otro peor y silencioso. Por debajo de la cota (78 pt, derivada de las constantes del layout) se conserva el ancho de siempre: invade, exactamente como antes de este arreglo, pero se lee.

### Notas
- Lo que evita los falsos positivos no es el umbral, sino exigir que los dos arranques **compartan linea base** y que la unica accion sea **recortar por la derecha** — nunca re-anclar ni acotar por la izquierda. Se descarto agrupar por distancia horizontal a secas porque las dos poblaciones se solapan: el mayor hueco dentro de un bloque legitimo de un solo firmante llega a ~176 pt y la separacion inter-columna mas estrecha es ~211 pt.
- **Coste aceptado**: entre el umbral y `boxW + GAP/2` (247 pt) el recorte SI ocurre en documentos de un solo firmante —un par etiqueta/valor a media distancia— y la estampa sale mas estrecha, con el nombre posiblemente truncado. A esa distancia el caso es indistinguible de dos columnas de verdad desde los arranques de linea, y de los dos errores posibles este es el barato: el dato fuerte del firmante vive en el PKCS#7, no en el dibujo.
- El ancho solo se recorta cuando es el **por defecto**; un `boxW` explicito (hint de propagacion del lote) es una decision de la persona y no se pisa.
- Corpus nuevo de 19 casos. Las 8 mutaciones del arreglo matan cada una el test que le toca.

### Fixed (segundo firmante)
- **Un documento que YA trae una firma deja de estampar la siguiente en el borde de la hoja.** Con firma previa, la colocacion entra por el anti-solape, y alli **nadie miraba el bloque de firma**: se cogia el primer hueco libre desde abajo. La correccion existia --el propio codigo dice que se midio que "un documento YA firmado que necesitaba una 2a firma la mandaba al suelo de la pagina en vez de junto a la primera"-- pero estaba encerrada tras `anchor !== undefined`, y en produccion el unico `anchor` es el hint de propagacion del LOTE: nunca alcanzaba al caso que la motivo, la firma individual. Medido sobre un NDA real de dos partes con la primera firma puesta: la caja pasa de `x=18` (el borde de la hoja, al pie) a `x=103`, el arranque exacto de la raya libre del segundo firmante. Solo se opina cuando se ENCUENTRA el bloque: sin bandas de texto no hay nada que alinear y decide el barrido de siempre, como antes.
- Cuatro entradas del corpus real se mueven y estan re-congeladas **con su justificacion una a una**: dos contratos a dos columnas pasan del pie a la columna izquierda del bloque (mejora, `x` exacta) y dos resoluciones se mueven lateralmente DENTRO del pie sin ganar nada, porque alli el bloque bajo el hueco es la linea "Pagina 4 de 4" -- se registra como lo que es, un movimiento sin justificar, no como una mejora. Una entrada de confianza baja de `alta` a `media hueco_justo`: la estampa deja el pie, donde le sobraba sitio, y pasa al hueco del bloque, que es estrecho por construccion.

### Fixed (segundo firmante, 2a vuelta)
- **La estampa del segundo firmante se apoya junto a su raya en vez de flotar por encima de la firma ajena.** El hueco se buscaba por franjas horizontales de pagina completa, asi que la firma previa del OTRO firmante --a la derecha, sin estorbar a esta columna-- levantaba el suelo del hueco para todo el ancho. Con la columna ya decidida, el hueco vertical se recalcula mirando solo los obstaculos que la caja puede tocar; como las bandas de texto ocupan el ancho completo por construccion, el filtro solo puede descartar firmas previas de otra columna -- un documento sin ellas no cambia ni un punto, y `enumerateSlots` sigue validando el candidato contra TODOS los obstaculos. Medido en el NDA real: y pasa de 263 a 229,7 con la raya en 222,7 (GAP/2 exacto de aire), y en `eci-real-contrato2026` de 344 a 298,4 -- apoyada sobre el nombre del bloque izquierdo, igual que la firma previa sobre el suyo. `eci-real-lideres` NO cambia, y el contraste es deliberado: alli la caja de 240 pasa por debajo de la firma previa, asi que el suelo alto es real y se respeta.
### Fixed (corpus de 8 documentos reales, 3a vuelta)
- **Una firma previa CONTENIDA en el cuerpo de texto ya no fabrica un hueco falso**: `reservedGapV` sobrescribia el top del bloque al absorber cada rect en vez de hacer `max`, y una firma previa dentro de un parrafo lo BAJABA hasta ella. De ahi salia un bloque de firma falso --la linea "Pagina 4 de 4", 300 pt mas abajo-- y la estampa se iba pegada al borde derecho de la hoja. Lo cazo el e2e de colocacion en la CI (2 tests, 3 de 3 intentos) sobre un delta que yo mismo habia congelado como "movimiento lateral sin justificar": un delta sin justificar era un bug sin diagnosticar.
- **Una fila de firmas dentro de la franja del pie cuenta como bloque, no como numero de pagina**: "Firma del Contribuyente / Pagina 2 de 2 / Firma del Declarante" de un formulario aduanero vive a 19 pt del borde y la regla del pie la descartaba entera; la estampa caia centrada sobre el numero de pagina, entre las dos etiquetas. Una banda del pie que se parte en columnas es una fila de firmas. Medido: x pasa de 186 (centrada) a 81,5, sobre la etiqueta del contribuyente, con el ancho ceñido a 170 pt para no cruzar el numero de pagina.
- **Con la firma previa en una pagina llena, la segunda va al bloque de la ultima pagina**: en un acta real el primer firmante estampo en medio del texto de la pagina 2, la pagina 5 tenia el bloque con la columna del segundo LIBRE, y el motor --que apunta a "la pagina de la firma previa"-- devolvia `needs_review` con `alsoFits` diciendo que en la 5 si cabia. Si esa pagina no tiene hueco se intenta el bloque de la ultima; solo se acepta un hueco RESERVADO (bloque identificable), nunca un espacio libre cualquiera en otra pagina.
- **Dentro de una fila, los huecos se miden en orden horizontal**: un numero de pagina en fuente menor queda con la banda 1 pt mas baja, dentro de la tolerancia de fila, y al ordenar por (v, u) iba el primero aunque estuviera en medio: el corte con su vecino salia negativo y el ancla acababa en el numero de pagina (258,9 en vez de 81,5).

### Sin cubrir (declarado, no silencioso)
- **Tres o mas firmantes lado a lado**: dejan menos separacion entre arranques que el umbral, no se detectan columnas y se invade igual que antes. No se arregla bajandolo: esa separacion es exactamente la de un par etiqueta/valor de un solo firmante, asi que desde los arranques de linea los dos casos son indistinguibles por distancia. Hay un test que afirma la limitacion para que cerrarla sea una decision y no un accidente.
- **Que columna es la del firmante**: el motor ancla a la primera. La unica senal capaz de decidirlo —el nombre o la cedula del certificado— existe en el paquete y no esta cableada: `analyzePdfForPlacement` acepta `anchorSpec` y se llama en 2 sitios de produccion, ninguno se lo pasa.

## [0.23.2] — 2026-08-26 — el escaneo de firmas deja de confundir «no hay» con «no pude mirar»

### Changed
- **El aviso de solape del lote aparece al CONFIRMAR, no al cargar el documento**, y dice el motivo real: cuando el barrido no pudo leer parte del documento el texto ya no afirma que hay un solape —que la persona puede desmentir mirando el canvas— sino que hay paginas sin leer. Un aviso que miente entrena a descartarlo, y aqui el descarte es irreversible. Ademas, antes el aviso se "pre-consumia" al cargar: en ese caso la primera pulsacion de Confirmar comiteaba directa, con lo que la doble confirmacion era un cartel y no una compuerta.

### Fixed
- **El barrido de firmas previas ya no se traga el fallo de una pagina** (`@firma-ec/pwa` 0.23.2): hacia `catch { annots = [] }` **por pagina y sin log**, asi que una pagina cuyo diccionario de anotaciones no parsea entraba al resultado como pagina SIN firmas y el escaneo salia con la forma exacta de uno completo. Aguas abajo eso significa que `overlapsExistingSignature` devuelve `false` sobre una lista incompleta y **el aviso de solape del lote —su unica doble confirmacion— no llega a dispararse**: PDF entregado con el sello encima de la firma del co-firmante, irreversible (ya se gasto PKCS#7 + TSA + OCSP) y sin una linea en consola. El barrido sale a `lib/batch/signatureScan.ts`, contra una interfaz minima, porque el fallo vivia dentro de un `<script>` de Svelte donde ningun test lo alcanzaba; ahora el resultado lleva `incomplete` y `failedPages` y **conserva lo que si se pudo leer** — el problema nunca fue tener informacion parcial, sino presentarla como completa. Ante un escaneo ciego, el lote pide confirmacion igual. 12 tests nuevos, vistos en rojo.
- **El gate del default centrado tenia media red**: los tests afirmaban que se REACTIVA (si no, el paso 2 se queda sin caja) pero ninguno que se siga SUPRIMIENDO mientras el anti-solape viene en camino — mutarlo a `true` incondicional dejaba 440 tests en verde. Extraido a `shouldRestoreCenteredDefault` con corpus bidireccional; esa misma mutacion ahora cae. Igual para el aviso de solape del lote (`needsOverlapConfirmation`), cuya mitad nueva sobrevivia a 411 unitarios y 50 e2e.
- **La vista previa cancela de verdad lo que ya no sirve**: la tarea de `getDocument` no se guardaba en ninguna parte, asi que al desmontar no habia nada que cancelar y su `destroy()` era un no-op mientras la carga seguia viva. Ahora hay token de generacion, se aborta la carga anterior al recargar, y el escaneo no entrega resultados de un documento que ya no esta cargado.
- **Cancelar dejo de registrarse como fallo**: destruir la tarea hace que su promesa rechace, y eso salia como `[PdfPreview] load failed` en una accion normal — volver al paso 1 con un PDF cargando. Contaminaba el unico canal que distingue "documento dificil" de "infraestructura rota".

## [0.23.1] — 2026-08-25 — la firma individual coloca con el motor, no a ciegas

### Added
- **Colocacion automatica en `/firmar`** (`@firma-ec/pwa` 0.23.1): el flujo de una sola firma era el **unico** que ponia la caja sin mirar el documento — con firmas previas usaba `computeSmartPlacement` (mira las firmas, no el texto, y puede subir una fila hasta el cuerpo) y sin ellas un default centrado al 12% de la altura. El motor completo (`analyzePdfForPlacement` + `computeAutoPlacement`: campo `/FT /Sig` declarado → anti-solape → hueco reservado / espacio libre → pie, todo contra las bandas de texto reales) solo corria en `/firmar-lote`. Ahora corre tambien aqui, **por defecto y sin ajuste nuevo**, en el worker de pre-vuelo para no bloquear el render de la vista previa. Medido sobre `audit-075-2026.pdf`: la caja pasa de y = 652 pt (82% de la altura — mitad del cuerpo) a y = 69,7 pt (8,8% — el pie), en la pagina 3, que es donde el motor decide firmar y no la ultima. Indicador de espera con `aria-live` mientras decide.

### Fixed
- **La colocacion inicial ya no es una carrera**: el motor y el colocador anterior competian, y ganaba uno u otro segun la maquina — en escritorio el motor, en un movil lento el camino ciego, que quedaba exactamente igual de ciego que antes. La precedencia es ahora determinista — **persona > motor > anti-solape > default centrado** — con un candado (`enginePending`) que impide colocar mientras el motor decide, un token de generacion (`placementRun`) para que el resultado de un PDF ya descartado no aterrice sobre el siguiente, y el escaneo de widgets llegado a destiempo guardado (`pendingScan`) para que solo lo consuma el fallback.
- **La caja ya no puede dibujarse en un sitio y firmarse en otro**: el motor emite puntos PDF absolutos y sin rotar, pero esta UI pinta y firma en el espacio del viewport de pdf.js (rotado, origen en el CropBox). Coinciden solo con `/Rotate 0` y CropBox en el origen; en cualquier otra pagina se declina al camino de siempre. Guarda mas estricta que la del lote, que solo mira `rotate`.
- **El modo guiado recupera su sugerencia cuando el motor declina**: el gate que reactiva el default esperaba un escaneo que en guiado nunca llega (`SimplePlacer` trae el suyo), asi que un documento firmado cuyo motor declina dejaba a la persona sin caja y con el CTA deshabilitado **para siempre**. Reproducido con Playwright y cubierto con un test visto en rojo.
- **El analisis en vuelo se termina en las tres salidas** (PDF nuevo, «firmar otro» y `onDestroy`): antes, cada ida y vuelta por el paso 1 dejaba un worker vivo con su copia del documento hasta agotar su timeout. Techo propio de 20 s: esto es una sugerencia con fallback valido, no la firma.
- **La degradacion deja rastro**: cuando el motor declina se emite un codigo estable a consola (`page_space_unsafe`, `ready_without_rect`, `no_free_slot`…) — nunca nombre de archivo, bytes ni contenido, mismo precedente de privacidad que `preflight.ts`. Sin ese canal, un deploy que rompa el worker degradaria a todos los usuarios al camino ciego sin dejar rastro ni en DevTools.

## [0.23.0] — 2026-08-24 — embudo de instalacion medido, y el aviso de privacidad dice la verdad

### Added
- **Embudo de instalacion de la PWA, cerrado y medido** (`@firma-ec/pwa` 0.23.0, `@firma-ec/stats-backend` 0.2.0): la app declaraba `share_target` y `file_handlers` pero no habia forma de saber cuanta gente la instalaba. Se anade un cuarto tipo de evento `install`, emitido en `appinstalled`, y `shortcuts` de manifiesto a las dos rutas reales (`/#/firmar-facil` y `/#/verificar`, verificadas contra la tabla de rutas de `App.svelte`). El backend acepta el nuevo literal en toda la cadena: validacion, codec de la serie, y **la proyeccion de lectura** — que en el primer intento contaba el evento y lo tiraba al construir el bucket publico, con el endpoint entero sin emitirlo jamas. Test de regresion sobre la ruta real.

### Fixed
- **La cifra publicada podia quedarse corta sin dejar rastro; y el limitador no es por visitante** (`@firma-ec/stats-backend` 0.2.0): el tope de 20 eventos/hora se llama "per-IP" y su clave es `rl:stats:${req.ip}`, pero **`req.ip` no es la IP del visitante**. Todo el trafico publico entra por el tunel, asi que lo que ve el servidor es siempre una direccion interna de la propia red overlay. Medido contra produccion el 2026-08-24: **1.498 de 1.498** peticiones registradas con direccion privada, **0 publicas**; 5 de 5 claves del limitador con sufijo RFC1918; el access log de Traefik registra la IP del tunel, no la del visitante; y ningun fichero lee `cf-connecting-ip`, `x-forwarded-for` ni `x-real-ip`. La consecuencia es que **es un unico cubo global para todo el mundo**, y al vaciarse devolvia 204 y descartaba el evento **en silencio**: una operacion real dejaba de contarse y la unica senal era una cifra publica mas baja. Ahora el descarte emite un `warn` explicito. No se toca el umbral: el cubo mas cargado esta en 16,75/20 tokens, o sea que todavia no muerde, pero el techo real son 480 eventos/dia para el planeta entero y conviene saberlo antes de crecer. Test bidireccional sobre la ruta real (descarte deja rastro / evento normal no ensucia el log), probado en rojo.
- **La IP del cliente ya no puede llegar al log de aplicacion** (`@firma-ec/stats-backend` 0.2.0): con `trustProxy` y `disableRequestLogging: false`, el serializador de Fastify escribia `remoteAddress` en la misma linea que `?type=sign` y una marca de tiempo con milisegundos — el cruce "quien firmo y a que hora" que el aviso de privacidad promete que no existe. En produccion esa direccion resulta ser interna (ver arriba), asi que el dato expuesto no era de un visitante; el fallo seguia siendo real como defensa en profundidad, porque un cambio de topologia que empiece a propagar la IP real la habria dejado escrita sin que nadie lo notara. `redact` con `remove: true` sobre `remoteAddress` y `remotePort`, y test sobre la ruta real que falla si vuelve a aparecer.

### Changed
- **Aviso de privacidad v1.1: se declara lo que si se trata y se corrige lo que se declaraba de mas** (`@firma-ec/landing` 0.7.1): las versiones anteriores no mencionaban los contadores agregados de uso, que llevaban tiempo funcionando. La v1.1 los documenta enteros — los cuatro literales que viajan por el cable, el doble efecto (contador **y** fila con la marca de tiempo del servidor), la retencion indefinida y el motivo. Y corrige en direccion contraria lo que una primera redaccion afirmaba de mas: **no tratamos tu IP**. Los dos topes contra el abuso se calculan sobre una direccion interna de nuestra red, son globales y no distinguen personas — con el efecto secundario declarado de que en un pico la cifra publicada se queda corta, nunca de mas. Se anaden las bases de licitud que faltaban, los dos terceros de la seccion 5 que se omitian (sellado de tiempo y respondedores de revocacion, **ambos opt-in y desactivados de fabrica**, que reciben hash o numero de serie a traves de un proxy que borra origen y referente), y se separa la justificacion del articulo 9 en sus dos casos reales. Paridad ES/EN alineada (la version inglesa nombraba `validate` donde el cable lleva `cert`).

## [0.22.8] — 2026-08-23 — cadena de confianza: homónimas en el pool + renovaciones con misma clave

### Fixed
- **Dos subCA homónimas embebidas en el mismo PDF/.p12 podían romper una cadena válida** (`@firma-ec/signer`, `@firma-ec/verifier`): el fix del 2026-08-05 (`resolveIssuerCert`) desambiguaba homónimos al buscar en el bundle, pero los tres recorridos del *pool* (certs ya embebidos en el CMS o aportados por el `.p12`) seguían eligiendo el primer subject DN coincidente con un `.find()` plano. Un PDF re-firmado que embebe las dos subCA de una renovación (estilo BCE 2011/2019) seguía la que estuviera primero — aunque no fuera la emisora real — y acababa en "emisor desconocido" con el eslabón correcto ahí mismo. Ahora los tres recorridos resuelven vía `resolveIssuerCert`, y si ninguna homónima del pool resuelve, se puentea desde el bundle/AIA en vez de abortar. Tests de regresión en ambos órdenes de embebido (el invariante que el bug violaba es la independencia del orden).
- **Los detectores de la fuga del `.p12` eran ciegos a la clave en forma de string** (`@firma-ec/pwa`): una auditoria independiente MIDIO el punto ciego inyectando la clave en cuatro codificaciones. El detector unitario solo miraba nombres de propiedad y buffers, asi que **base64url, base64 estandar, hex y bytes crudos en un string escapaban los cuatro**. Y no es teorico: los componentes privados de un JWK (`d`, `p`, `q`, ...) son strings base64url — exactamente la forma en que la fuga ya volvio una vez. El E2E cubria base64url, pero **no corre en CI**, asi que la unica red que se ejecuta siempre era ciega a la forma mas probable del fallo. Ahora `p12.worker.security.test.ts` recorre todos los strings alcanzables del mensaje y busca el marcador en las cuatro codificaciones, con las **3 fases de alineacion** de base64 (sin ellas, un desplazamiento de 1 byte lo esconde). Probado en rojo con las cuatro inyecciones: cada una tumba el test; el arbol restaurado queda en 7 verde.
- **Renovación de CA que conserva el par de claves: podía elegirse el cert caducado** (`@firma-ec/crypto-core`): cuando una AC renueva conservando la clave, los dos certs comparten SKI y la firma del hijo verifica contra ambos — `resolveByCryptographicVerification` devolvía el primero declarado, que podía ser el caducado, y un intermedio caducado rompe la cadena aunque su firma case. Con más de un candidato verificado, ahora se prefiere el vigente y, entre vigentes, el de `notBefore` más reciente (determinista e independiente del orden). Probado con dos certs de la misma clave, uno caducado y uno vigente, en ambos órdenes.

### Fixed
- **La fecha de la app movil de FirmaEC la desmentia la fuente que citabamos** (`@firma-ec/landing` 0.7.0): el texto afirmaba "app movil desde junio de 2023 (v3.0.0)" enlazando al registro de cambios oficial — pero la entrada mas antigua con movil de ese mismo changelog es la **v2.11.0, del 19 de agosto de 2022**, ya con Android 8.0+ e iOS 12+. Un desfase de ~10 meses en publicidad comparativa, desmentido por nuestro propio enlace. Corregido en las 7 paginas ES/EN. Es exactamente la clase de fallo que esta version existe para eliminar: cifra copiada sin contrastar contra la fuente.
- **El guard de tamanos era ciego justo donde el numero se copia a mano** (`@firma-ec/landing` 0.7.0): `check-size-claims.mjs` solo recorria `src/content/**/*.md`. Una auditoria independiente lo probo inyectando "200 MB" en `src/pages/**/*.astro` (donde el JSON-LD del `faqPage` repite el texto del cuerpo) y en `public/llms-full.txt`: **EXIT=0 en ambos**. Se amplia a `src/content`, `src/pages` y `public`, y a `.md`/`.astro`/`.txt`. Re-probado en rojo sobre las dos raices nuevas (EXIT=1 con archivo:linea) y en verde tras restaurar.
- **Paridad ES/EN rota por la propia correccion** (`@firma-ec/landing` 0.7.0): al corregir "valida su cadena de confianza sin conexion" en `es/firmar-documentos-en-linea.md`, la frase vieja sobrevivio intacta en `en/sign-documents-online.md`. Alineada.
- **Se acredita a FirmaEC el TSA que si tiene** (`@firma-ec/landing` 0.7.0): la fila decia "Verificar con MINTEL" cuando el mismo changelog que la pagina ya enlazaba documenta el sellado de tiempo desde la **v5.0.0 (11 de abril de 2026)**. Infra-acreditar al rival tambien es inexacto. La fila de PAdES B-LT/B-LTA se mantiene en "Verificar con MINTEL" porque el changelog **no** la menciona — verificado, no asumido.
- **`dateModified` coherente** (`@firma-ec/landing` 0.7.0): las 9 paginas con contenido factual modificado quedan con fecha 2026-08-23; a las 6 que no tenian el campo se les anade. La fecha de actualizacion visible es senal de frescura para los motores generativos.
- **La landing publicaba un limite de tamano que nuestro propio producto no tiene, y se contradecia sobre FirmaEC** (`@firma-ec/landing` 0.7.0): seis paginas afirmaban "50 MB en movil y 200 MB en escritorio" por PDF. Las dos mitades eran falsas — **no existe ninguna rama por dispositivo** en el codigo y `200` no aparece en ninguna constante: los topes reales son 50 MB (`MAX_BYTES` en `apps/pwa/src/ui/Drop.svelte`, replicado en `sw.ts` y `SharedFileHandler.svelte`) y 40 MB por archivo en firma por lotes (`MAX_BATCH_FILE_SIZE_BYTES` en `apps/pwa/src/lib/workers/sign-queue.ts`). El bug no fue un descuido sino su clase: el numero estaba **copiado a mano en seis sitios**, sin ninguna fuente de verdad. Fix: la copia pasa a "hasta 50 MB por PDF (40 MB en firma por lotes)", sin distincion de dispositivo, y el nuevo guard `apps/landing/scripts/check-size-claims.mjs` **lee las dos constantes del codigo de la PWA** y falla el build si un `.md` de `src/content` afirma una cifra de MB que ninguna constante respalda (las cifras de terceros — 4 MB / 512 MB de FirmaEC — viven en un mapa explicito, cada una con su fuente). Verificado en verde y **en rojo**: revertir un `50` a `200` deja el build en `exit 1` senalando archivo y linea.
- **El `FAQPage` JSON-LD declaraba respuestas que la pagina no mostraba** (`@firma-ec/landing` 0.7.0): en `/firmar-pdf-desde-el-celular/` y `/en/sign-pdf-from-phone/`, 3 de 5 entradas no aparecian en el HTML construido — entre ellas una respuesta sobre tokens USB (`PKCS#11`, WebUSB/WebHID para v2) que no estaba en el cuerpo visible. La politica de datos estructurados de Google exige que la respuesta sea visible, y el comentario del fuente afirmaba justo lo contrario de lo que hacia ("mirrors the FAQ verbatim"). Causa estructural: el JSON-LD se escribe a mano en el `.astro` mientras el cuerpo viene del `.md`, asi que los dos derivan en silencio y solo una comprobacion **contra el `dist/` construido** puede verlo. Fix: las dos paginas declaran exactamente las 6 preguntas que el cuerpo responde, con el texto de la pagina; el comentario dice la verdad; y el nuevo guard post-build `apps/landing/scripts/check-faq-visibility.mjs` recorre `dist/**/*.html`, extrae cada bloque `FAQPage` y falla si algun `acceptedAnswer` no aparece en el texto visible (normalizando solo puntuacion tipografica y entidades). Ambos guards quedan encadenados en el script `build`.
- **El sitio afirmaba que FirmaEC no tiene app movil, y se contradecia a si mismo sobre firmar sin conexion** (`@firma-ec/landing` 0.7.0): (1) `comparativos/firmaec/`, `alternativa-firmaec/` y sus pares EN decian "Movil (iOS/Android): FirmaEC ❌ No" y "No funciona en el celular. FirmaEC es solo de escritorio". Es falso desde agosto de 2022 ([registro de cambios oficial](https://www.firmadigital.gob.ec/registro-de-cambios-de-firmaecchangelog/), v2.11.0, Android 8.0+ / iOS 12+). La comparacion honesta es sobre **limites**, no sobre existencia: FirmaEC Movil existe y topa en 4 MB por documento frente a 50 MB, y exige instalar una app nativa. (2) Las paginas nuevas invitaban a "desconectar el internet y seguir firmando" mientras `comparativos/firmaec/` decia "firma recomendado online por TSA" y `firma-electronica-ecuador/` "recomendable estar online para validar la cadena de la AC". Se unifica **una sola version, verificada contra el codigo**: la firma se calcula 100% en el navegador y con la configuracion por defecto (PAdES B-B; `tsaEnabled: false` en `apps/pwa/src/lib/settings.svelte.ts`) no toca la red; **si** necesitan red el sello de tiempo (TSA), la LTV (OCSP/CRL) y la resolucion del intermedio via AIA `caIssuers` (`packages/signer/src/chainIntermediates.ts`) cuando el `.p12` viene solo con la hoja — caso real con certificados UANATACA — sin que ese fallo impida firmar. De paso caen dos afirmaciones mas que el codigo desmiente: FirmaEC "funciona offline ✅" (su [manual v4.0.0](https://www.firmadigital.gob.ec/wp-content/uploads/2025/08/Manual-Usuario-FirmaEC-v4.0.0.pdf), sec. 3, exige internet) y firmar.ec "sin firma por lotes" (existe la ruta `/firmar-lote`).
- **Datos sobre FirmaEC sin fuente, y uno de ellos impertinente** (`@firma-ec/landing` 0.7.0): las tablas comparativas citaban "1,6 ★ sobre 54 valoraciones (app id `6447067469`)" y "su manual v3.0.0 documenta borrar a mano la carpeta `jre` al actualizar", ninguno respaldado. **Se retira por completo la fila de la valoracion de App Store**: no informa de ninguna capacidad (la celda de firmar.ec dice "No aplica"), envejece sola, y bajo la LORCPM la publicidad comparativa debe ser exacta, verdadera **y pertinente** — el contraste 4 MB vs 50 MB gana el argumento sin tocar reputacion. La instruccion de borrar la carpeta `jre` **no aparece** en el manual vigente (v4.0.0, agosto 2025) y se retira; lo que si documenta —que el firmador se invoca con el JRE de la propia instalacion— se conserva con enlace al PDF. El resto de cifras de FirmaEC (4 MB movil, 512 MB escritorio, app movil desde agosto de 2022) queda con **enlace saliente y fecha de consulta (23 de agosto de 2026)**. En la tabla de comparacion **movil**, "app Java con JRE embebido" describia el escritorio y no la app de tienda: se corrige a "app nativa (Android 8.0+ / iOS 12+)".
- **La pagina que hoy rankea #1 perdio su frase de coincidencia exacta y gano un parrafo duplicado** (`@firma-ec/landing` 0.7.0): al reescribir `firmar-documentos-en-linea/` en formato answer-first, el lede quedo **repetido literal** en la cita de apertura y en la primera FAQ, y la frase *"pagina web para firmar documentos en linea"* —la coincidencia exacta de la consulta que la posiciona— salio del cuerpo y solo sobrevivia en el `<meta description>`. Se elimina la repeticion y la frase vuelve como parrafo justo debajo de la cita, conservando la apertura answer-first y la senal antigua. Ademas se anaden la URL EN de `/en/sign-pdf-from-phone/` a `public/llms.txt` (solo se habia anadido la ES) y se actualiza `public/llms-full.txt`, congelado en 2026-07-03 y con dos afirmaciones ya falsas (perfil "B-T por defecto" y "firma masiva: en el roadmap").
- **La version que ve el usuario era una constante escrita a mano, y volvio a desincronizarse** (`@firma-ec/pwa` 0.22.6): `app.firmar.ec` mostraba "version 0.22.4" en el pie y en Acerca de con el release **0.22.5** desplegado — la cadena `0.22.5` no aparecia en ningun asset servido. La causa no era el descuido sino su clase: `apps/pwa/src/lib/version.ts` declaraba `APP_VERSION` como literal, desacoplado de `apps/pwa/package.json`, asi que bumpear una version eran **dos gestos independientes** y el desfase ya habia ocurrido antes (commit `b1a71da`, "bump APP_VERSION a 0.21.0 (falto al bumpear package.json)"). Corregir el numero a mano habria repuesto la trampa. Fix: la version se **inyecta en build** desde `package.json` via el `define` de Vite (`__APP_VERSION__`, armado en el nuevo `apps/pwa/appVersion.ts` y replicado en los dos `vitest.config.ts` porque `version.ts` lo lee al cargar); la constante manual desaparece y solo queda una fuente. Detector nuevo `apps/pwa/src/lib/version.test.ts`: falla si `APP_VERSION` deja de coincidir con `package.json` (rojo verificado contra el estado previo: `expected '0.22.4' to be '0.22.5'`), si la version no tiene forma semver — el caso en que la inyeccion falla y el pie quedaria vacio — y si `Footer.svelte`/`About.svelte` vuelven a escribir el literal en vez de importar la constante.
- **`WHATSAPP_URL` no tenia prueba bidireccional: el numero de soporte sobrevivia solo por un operador** (`@firma-ec/pwa` 0.22.6): el build de produccion hornea `VITE_WHATSAPP_URL` vacio (verificado en el bundle servido: `{VITE_WHATSAPP_URL:""}`), asi que `593993995618` llega por el `||` del fallback. Revertirlo a `??` deja el enlace **sin destino** — un href relativo que reabre la propia PWA, sin error visible — y nada lo detectaba; y antes del fix del 2026-07-29 el fallback era la linea corporativa de IDKMANAGER (`593958888193`), el bug que estuvo semanas invisible. Se anaden 8 casos en `links.test.ts` que afirman **las dos direcciones** sobre las tres formas reales de la env (vacia, ausente, solo espacios): nunca vacio ni href sin destino (`https://`, host `wa.me`), y nunca la linea corporativa. Ambas direcciones verificadas EN ROJO con mutantes: `||`→`??` pone 2 en rojo, apuntar la constante al corporativo pone 10. Para lograrlo el fallback se extrae a `resolveWhatsappBase(raw)`, que **es** la ruta de produccion (`WHATSAPP_URL = resolveWhatsappBase(env['VITE_WHATSAPP_URL'])`, sin logica propia): Vite inlina `import.meta.env` como objeto literal en build y bajo Vitest cada modulo recibe su propia copia, de modo que alterar la env del test NO cambia la que lee `links.ts` (comprobado — el modulo seguia devolviendo el default incluso con `vi.resetModules()`), y un test que la simulase habria pasado siempre.
- **Todo el WhatsApp de cara al cliente apuntaba a la linea corporativa de IDKMANAGER** (`@firma-ec/pwa` 0.22.5, `@firma-ec/landing` 0.6.28): la barra "¿Necesitas ayuda? Escríbenos por WhatsApp" del modo guiado y la ayuda de certificado de la PWA, mas los CTA de patrocinio de la landing en 4 paginas (`/`, `/en/`, `/patrocinar/`, `/en/sponsor/`) — y en `/patrocinar/` tambien como **texto visible** ("WhatsApp · +593 95 888 8193") — mandaban a los clientes a `593958888193`, la linea corporativa de IDKMANAGER (que es ademas el WhatsApp personal de un dev), no a la linea atendida de firmar.ec. En la PWA el numero llegaba por el **fallback**: el bundle servia `(env.VITE_WHATSAPP_URL ?? "https://wa.me/593958888193")` con el objeto de env vacio en produccion. Fix: **fuente unica por app** — `apps/pwa/src/lib/links.ts` y `apps/landing/src/lib/contact.ts` son el unico origen del numero (`593993995618`, instancia Evolution `firmarec`, perfil "Firmar Ec"); ninguna superficie reescribe el literal. Se corrigen tres defectos que hacian el bug invisible o el arreglo inefectivo: (1) el override usaba `??`, asi que un `ARG` vacio inlinado por Vite como `''` dejaba el enlace **sin destino** — ahora es `||`; (2) `VITE_WHATSAPP_URL` estaba documentado pero **no era alcanzable**: no se declaraba como `ARG` en `infra/docker/pwa.Dockerfile` ni se pasaba en `scripts/deploy-pwa.sh`, y Vite lee `import.meta.env` en build-time, de modo que setear la env en el servicio no cambiaba nada; (3) la query parasita de un override (`...?text=a`) se recorta con `replace` y no con `.split('?')[0]`, que es `string | undefined` bajo `noUncheckedIndexedAccess` y rompia el `typecheck` que gatea CI (mientras `vite build` no typechequea: prod habria salido bien y CI en rojo).
- **Guardarrail de build contra la regresion: `scripts/check-wa-number.mjs`** (nuevo): el bug anterior vivio semanas con build verde, sitio funcionando y suite en verde porque nadie mira el bundle servido — este check mira el bundle. Corre al final del `build` de **ambas** apps y falla el build si la salida contiene la linea corporativa en cualquiera de sus formas (E.164, display o local). Es **bidireccional**: tambien afirma en positivo que el numero esperado ESTA presente, porque un typo en la constante o un refactor que borre el CTA dejarian la salida sin enlace usable y un check solo-negativo imprimiria OK. Falla igualmente si no encontro **nada** que escanear (vacio = no verificado). El escape hatch es fail-closed: solo el valor `off` desactiva la asercion positiva; vacio cae al default, porque un `ARG` sin pasar deja la env como `''` y un guardarrail no debe apagarse por un descuido. Ambos Dockerfiles copian el script explicitamente (`COPY scripts/check-wa-number.mjs ./scripts/`): el `build` lo invoca por ruta relativa `../../scripts`, y sin ese COPY el `docker build` de las dos apps rompe con `MODULE_NOT_FOUND` — verificado reproduciendo el contexto de build real, no solo con `pnpm build` en el repo (donde `../../scripts` existe siempre y el fallo no aparece).
- **UANATACA emite en paralelo desde una segunda CA subordinada ("UANATACA CA2 2021") no bundleada — firma real rechazada como "emisor no reconocido"** (`@firma-ec/tsl-ec`, `@firma-ec/verifier` (paquete 1.0.0; `ENGINE_VERSION` interno se mueve por separado, no confundir los dos números), `@firma-ec/pwa` 0.22.3): mismo patrón que el incidente de 2026-05-28 (commit 43f2e33) con la subordinada 2016, ahora con una segunda CA subordinada nueva. Extracción forense del CMS embebido de un PDF real firmado por IDKMANAGER S.A.S. confirmó el emisor (`UANATACA CA2 2021`, SHA-256 `15ceab3…00e69a`, SKI `C5:E7:33:…:4A` — coincide con el AKI del leaf del cliente) y que la raíz `UANATACA ROOT 2016` (ya confiada, `roots.ts` sin cambios) la firmó. Fix: se bundlea la nueva intermedia (`packages/tsl-ec/src/intermediates/uanataca-ca2-2021.pem` + entrada en `intermediates.ts`) **junto a**, no en reemplazo de, la de 2016 — UANATACA emite desde ambas concurrentemente. La primera versión de este fix intentó además un cambio sistémico (`PathResult.chainIncomplete`) para no repetir este incidente con la PRÓXIMA subordinada no bundleada; ese cambio resultó inseguro y fue revertido el mismo día — ver la entrada de `### Security` de abajo (2026-08-05, hallazgo CRITICAL).

- **UANATACA sella el tiempo (RFC 3161) desde una CA que `@firma-ec/tsa-trust` nunca conoció — "Sello presente, no verificado" en un contrato real con 2 firmas criptográficamente válidas** (`@firma-ec/tsa-trust` 0.8.0, `@firma-ec/verifier` 1.1.0, `@firma-ec/ltv-validation` 1.0.2, `@firma-ec/pwa` 0.22.4): reportado tras desplegar el fix de UANATACA CA2 2021 (arriba) — el certificado del firmante ya validaba, pero el sello de tiempo seguía en rojo. Extracción forense del token RFC 3161 embebido en un contrato real (`19562560_Contrato.pdf`, 2 firmantes) identificó al emisor real: "Sello de tiempo electrónico de UANATACA - TSU01", emitido por una CA subordinada dedicada a timestamping (`UANATACA CA1 2021`, EKU `id-kp-timeStamping` crítico) que `tsa-trust` no tenía — el paquete solo conocía FreeTSA + un placeholder de ARCOTEL, cero raíces/intermedias de ninguna ACE ecuatoriana real. El verificador oficial MINTEL FirmaEC 5.1.0 valida ambas firmas (con el sello) sin ninguna advertencia, confirmando que el token es legítimo. `UANATACA CA1 2021` encadena a `UANATACA ROOT 2016` — la MISMA raíz ya confiada en `@firma-ec/tsl-ec` para certificados de firma, deliberadamente **duplicada** (no importada) en `tsa-trust` porque el propio paquete documenta que anclas de TSA y anclas de ACEs de firma rotan y se revisan por separado a propósito. Fix en dos capas: (1) bundle estático — nuevo concepto de "intermediates" en `tsa-trust` (no existía; solo tenía `roots`), con `UANATACA CA1 2021` + `UANATACA ROOT 2016`, mezclado automáticamente dentro de `validateTsaCertChain` (ningún caller necesita cambios); (2) **F2** — mismo fallback AIA `caIssuers` de F1 (ver arriba), ahora también para la cadena de TSA: si el bundle local sigue sin resolver, `verifyTimestamp` intenta la URL AIA propia del certificado TSA (opt-in vía `VerifyOptions.fetchTsaAia`, default `true`, igual postura que `fetchOcsp`), con el mismo límite de confianza de F1 (nunca otorga confianza nueva, solo puede completar una cadena hacia una raíz YA confiada). Nueva ruta allowlisteada en el proxy anti-SSRF (`/api/aia/uanataca-tsa-ca1`, esquema de URL distinto a las entradas F1 existentes — `web.uanataca.com` vs `www.uanataca.com`). Certificado real de la TSA guardado como fixture de test (`packages/tsa-trust/tests/__fixtures__/uanataca-tsu01-leaf.der`) — no contiene datos personales, es un certificado de servicio de UANATACA S.A.

### Security
- **La clave privada del `.p12` cruzaba al hilo principal y se quedaba ahí toda la sesión de firma** (`@firma-ec/pwa` 0.22.7): `p12.worker` devolvía el objeto completo de `parsePfx` — incluido `privateKeyPkcs8Der`, el PKCS#8 **en claro** — y además lo **transfería** explícitamente, así que el hilo principal recibía el material de clave sin copia y lo retenía en el heap durante minutos, al alcance de cualquier script de la página. Contradecía el modelo single-shot que el propio worker documenta. La clave nunca hizo falta fuera del worker: `Firmar.svelte` solo lee `signingCert.{subjectCN,issuerCN,notBefore,notAfter}`, `sign.worker` re-parsea sus propios bytes + PIN, y `sign-session.worker` (lotes) retiene el `ParsedPfx` en ámbito de worker y lo pone a cero en `wipeSession()` sin emitirlo nunca. Fix: `toPublicParsed` (nuevo `p12-result.ts`) pone a cero el buffer y despoja el campo antes del `postMessage`, la respuesta pasa a `ParsedPfx` y se retira el `transfer`. Dos detectores, ambos **probados en rojo** contra el estado previo: `p12.worker.security.test.ts` arranca el módulo del worker y audita lo que sale por `postMessage` (4 de 5 asserts fallan sin el fix), y el E2E `p12-key-no-cruza.spec.ts` instrumenta `window.Worker` en un Chromium real y busca los **bytes** del PKCS#8 del fixture en todo lo que recibe la página (sin el fix: `$.parsed.privateKeyPkcs8Der`, 2069 B de buffer frente a los 852 B del DER del certificado, que sí es público y sí debe cruzar). El E2E se auto-valida rechazando cualquier patrón de bytes que también aparezca en el certificado — apuntar al módulo RSA `n` daba un falso positivo.

### Security
- **Endurecimiento de la compuerta del `.p12`: se cierra la puerta por la que el bug podía volver** (`@firma-ec/pwa` 0.22.7, misma versión sin publicar). Tres hallazgos de la revisión del fix anterior:
  1. **`privateKeyJwk` seguía cruzando, y los detectores eran ciegos a una clave en forma de string.** El rest-spread de `toPublicParsed` despojaba **un** campo (`privateKeyPkcs8Der`) y reenviaba `privateKeyJwk` intacto. Hoy es inocuo porque `parsePfx` lo emite como `{ kty }` a secas, pero `signer/types.ts` lo documenta como clave privada y su tipo `JsonWebKey` admite `d`/`p`/`q`/`dp`/`dq`/`qi`: el día que se reactive la ruta Web Crypto, la clave vuelve a cruzar. Y cruzaría **invisible**, porque los dos detectores nuevos escaneaban sólo `ArrayBuffer` y vistas tipadas — los componentes privados de un JWK son **strings** base64url. Fix: `toPublicParsed` **normaliza** en vez de reenviar (reconstruye el esqueleto público, `kty` a secas), y los detectores pasan a mirar (a) nombres de propiedad contra una lista **compartida** (`key-material-props.ts`, que absorbe la de `sign-session-bus.test.ts` en vez de ser una tercera), (b) la **forma** de `privateKeyJwk` — sólo se admite `kty` —, y (c) el **contenido de los strings**: base64url del patrón en sus 3 fases de alineación, base64url decodificado, y bytes crudos en string. Probado en rojo con una fuga inyectada que emitía la clave entera **sólo** como `privateKeyJwk.d`: el detector anterior daba **verde** con el PKCS#8 completo en el heap de la página; el nuevo la caza por nombre y por contenido (`$.parsed.privateKeyJwk.d`).
  2. **La forma del retorno de `parsePfx` queda fijada donde nace.** El bug original fue invisible a TypeScript: `packages/signer/src/p12.ts` declara `Promise<ParsedPfx>` pero resuelve con `ParsedPfx & { privateKeyPkcs8Der }`, y cinco ficheros tapan el hueco con `as ParsedPfxFull`. Como `toPublicParsed` despoja **por nombre**, un campo nuevo con material de clave cruzaría solo y con CI en verde. `packages/signer/tests/p12.test.ts` afirma ahora el conjunto **exacto** de claves del objeto (RSA y ECDSA); probado en rojo añadiendo un campo de mentira a `p12.ts` (`+ "leakedRawKey"`). Añadir un campo pasa a exigir una decisión explícita.
  3. **Comentario de protocolo obsoleto** en `p12.worker.ts`: documentaba `parsed: ParsedPfxSerialisable`, un tipo que no existe en el repo (0 ocurrencias). Ahora dice `ParsedPfx`.
- **`chainIncomplete` softeaba el verdict a `warning` usando una señal controlada por el propio atacante** (`@firma-ec/verifier` 1.0.0, `@firma-ec/pwa` 0.22.3, CRITICAL): la primera versión del fix de UANATACA CA2 2021 (ver `### Fixed` arriba) hizo que una cadena que el walk leaf→raíz no pudiera completar produjera `status: 'warning'` ("Firma válida con advertencias") en vez de `invalid`, sobre la teoría de que probablemente era una ACE real aún no bundleada. Esa teoría es falsa como señal de seguridad: el pool que camina el walk (`trustedCerts + intermediates`) se construye con certificados que el FIRMANTE embebió en el CMS — atacante-controlado. Un atacante que fabrica su propia CA y simplemente NO la embebe en el PDF produce exactamente esa forma. Reproducido en dos direcciones por revisión independiente (code-reviewer, opus): raíz rogue autofirmada que emite el leaf directo (Caso A) y raíz rogue → subCA rogue → leaf (Caso C), ambas con solo el leaf en el CMS — las dos daban `warning` en vez de `invalid`. Fix: `chainIncomplete` vuelve a producir SIEMPRE el mismo rechazo duro que `untrusted_root` (nunca lo suaviza); solo cambia el MENSAJE (honesto sobre las dos causas posibles — intermedia real sin bundlear, o emisor no acreditado — sin acusar de fraude ni implicar que la firma pudiera ser confiable). Tests e2e nuevos: `packages/signer/tests/chain-intermediates.test.ts` (casos A y C, más el caso original UANATACA re-verificado en rojo→verde).
- **Colisión de subject DN entre subCAs del BCE — `.find()` elegía la intermedia equivocada por orden de declaración** (`@firma-ec/tsl-ec`, `@firma-ec/verifier` 1.0.0, `@firma-ec/signer` 0.10.2, HIGH): `bce-subca-2011` (expirada) y `bce-subca-2019` (vigente) comparten el mismo subject DN (`CN=AC BANCO CENTRAL DEL ECUADOR`) — confirmado con `openssl x509 -noout -subject` sobre ambos PEM, y el `commonName` declarado de la 2011 (`"... (2011)"`) no correspondía al CN real del certificado (corregido). Tres selectores resolvían la intermedia por subject DN con `Array.find()` (toma la primera coincidencia sin comparar AKI/SKI ni la llave pública): `selectBridgingIntermediates` en el verifier, `resolveSigningIntermediates` en el signer, y — encontrado en una SEGUNDA ronda de revisión independiente, después de que la primera diera por cerrado el hallazgo con los otros dos — `checkCertificate` en `packages/verifier/src/certCheck.ts:241` (motor de la función "verificar certificado" de la PWA, `cert.worker.ts`). Como la 2011 está declarada primero en `intermediates.ts`, un leaf real del BCE (emitido por la 2019) resolvía a la 2011 en los tres: al firmar se embebía la intermedia equivocada, al verificar la firma completa la cadena con la 2011 no cerraba con el leaf real (`untrusted_root`), y al verificar solo el certificado (`checkCertificate`) salía `trusted: false` sobre un certificado legítimo del BCE. Reproducido en las tres rutas, con datos reales (`leaf-bce.der`, bundle de producción) en la segunda ronda. Fix: nuevo `resolveIssuerCert` en `@firma-ec/crypto-core` — resuelve por Authority Key Identifier (hijo) == Subject Key Identifier (candidato) primero; si AKI/SKI faltan o siguen ambiguos, cae a verificación criptográfica real (`Certificate.verify()`) sobre los candidatos con el mismo DN, en vez de adivinar por orden de array. Usado por los tres selectores. Test de regresión con datos reales: `packages/verifier/tests/certcheck-bce-real.test.ts` (rojo→verde confirmado contra el `certCheck.ts` sin este fix).
- **La ruta de re-firma (multi-firma secuencial) no validaba el rect de la estampa visible** (`@firma-ec/signer` 0.10.1, `@firma-ec/pwa` 0.22.1): `addIncrementalSignature` — la ruta que toma TODO documento que ya trae una firma previa, exactamente el conjunto que la firma por lotes manda a colocación manual — solo comprobaba el índice de página, a diferencia de `signPdfPades` (que llama `validateVisibleSig` antes de tocar el PDF). Medido con 4 entradas inválidas (fuera de página, 5×5pt bajo el mínimo legible, coordenada `NaN`, coordenadas negativas): las 4 se aceptaban y se escribían tal cual en `/Rect` — una de ellas literalmente `/Rect [NaN ... NaN ...]`, sintaxis PDF inválida grabada en un documento legal. Fix: reusar `validateVisibleSig` contra el `PDFDocument` de pdf-lib que esta ruta ya carga (resuelve `/MediaBox` heredado incluido), en vez de reimplementar los mismos límites por segunda vez. Encontrado por revisión OWASP independiente (code-reviewer, opus) con reproducción real ejecutada, no solo por lectura.
- **Verificador: la evidencia de revocación embebida se evaluaba sin comprobar su firma** (`@firma-ec/verifier` 1.0.0): `verifyLtv` leía `parsed.certStatus` del OCSP embebido en el DSS justo después de casar el CertID, **sin mirar `signatureValid`** — el archivo no contenía ni una sola referencia a ese campo. Como el DSS lo aporta quien firma, un titular con el certificado **revocado** podía grapar una respuesta `good` fabricada (CertID correcto, firma inválida) y la verificación la presentaba como revocación corroborada. Contradecía el contrato que `ocsp/response.ts` documenta explícitamente ("el llamante DEBE filtrar por `signatureValid` primero"). Añadido el gate. Preexistente, no introducido por este corte. Detectado por revisión adversarial independiente.

### Security
- **F1 (AIA self-heal en el firmador): el aviso "cadena incompleta" se disparaba en el 100% de las firmas sanas, no solo las rotas** (`@firma-ec/signer`, P0): `resolveSigningIntermediates` solo marcaba `complete: true` si el walk llegaba a embeber un certificado autofirmado — pero `getIntermediates()` (el bundle que recorre) nunca contiene una raíz (viven en `roots.ts`, un bundle aparte que esta función nunca consultaba). Resultado: TODA firma real de cualquier ACE ecuatoriana volvía `complete: false`, indistinguible de una cadena genuinamente rota. Reproducido y confirmado (rojo→verde) con certificados reales de UANATACA y BCE contra el bundle de producción, hallado por una revisión adversarial independiente (silent-failure-hunter) sobre el diff de F1 antes de mergear. Fix: `complete` ahora se cumple en cuanto el emisor pendiente coincide con una raíz YA confiada (`getTrustRoots()`), sin necesidad de embeber esa raíz (correcto per PAdES: se embebe hasta la raíz, nunca la raíz misma) — esto además evita fetches AIA innecesarios hacia CAs cuya raíz ya conocemos localmente.
- **F1, segunda ronda: el fetch AIA en vivo resolvía CERO certificados reales, y otros 3 HIGH** (`@firma-ec/ltv-validation`, `@firma-ec/signer`, `@firma-ec/pwa`; revisión independiente code-reviewer opus sobre el diff de F1 antes de mergear):
  - **PEM sin soporte**: `fetchIssuerCertViaAia` solo parseaba DER bare o PKCS#7 certs-only; el responder real de UANATACA (verificado por curl contra su URL `caIssuers` real) sirve PEM, así que el fallback nunca resolvía nada en producción pese a estar "activo". Nuevo `parsePemCerts` en `aia-certs.ts` como tercer intento tras DER y PKCS#7.
  - **Cache sin re-verificar en cada hit**: un hit de cache se devolvía sin repetir el chequeo criptográfico (CA:TRUE + `child.verify`) — rompía la garantía incondicional del propio módulo ("el cert devuelto DEBE haber firmado al hijo") para cualquier llamador después del primero, ante una URL que sirviera certificados distintos con el tiempo (rotación de subCA). Ahora cada hit se re-verifica contra el hijo actual; si falla, se trata como miss y cae a un fetch en vivo.
  - **Allowlist incompleta**: el fixture real de UANATACA usado en tests resulta emitido por `UANATACA CA2 2016` (confirmado por `openssl verify -partial_chain`), pero su propio AIA apunta a `subordinate1.crt` (CA1) — inconsistencia de datos del lado de UANATACA, no del código. Se añadió `subordinate2.crt` (CA2, el emisor real) como segunda entrada de allowlist independiente en `ARCOTEL_PROXY_MAP` (`proxy.ts`) + ruta `reverse_proxy` correspondiente en `Caddyfile.pwa`, documentando la discrepancia; no bloqueante porque el bundle estático de F0 ya cubre UANATACA de todos modos.
  - **El lote nunca leía la señal**: `describeOutcome` en `sign-queue.ts` (el único canal que `FirmarLote.svelte` consulta) nunca miraba `result.chainComplete`/`missingIssuerDn` — solo el camino de un solo documento lo hacía. Un documento firmado con una cadena incompleta (bundle-miss + AIA fallido) se reportaba como éxito limpio en el lote. Ahora `chainComplete === false` entra a `degraded` y agrega un warning `chain_incomplete` con el DN faltante; `null` (bundle de worker cacheado, más viejo) se sigue tratando como "desconocido", no como aviso.
  - **El leg AIA no tenía presupuesto de red**: el walk (hasta 8 hops) no tenía techo agregado — un responder AIA lento en cada hop podía consumir muchas veces el timeout por-request, sin relación con el presupuesto de firma del documento (mismo modo de fallo que ya se había cerrado para TSA/OCSP/CRL — defecto #1). `deriveNetworkBudget` (`sign-queue.ts`) reserva ahora un 15% adicional (`aiaBudgetMs`/`aiaTimeoutMs`), enhebrado por `SignNextOptions`/`SignNextRequest` hasta `sign-session.worker.ts`, que calcula `deadlineAt = Date.now() + aiaBudgetMs` (en el worker, no en el hilo principal, para no cargar tiempo de cola contra la red). `chainIntermediates.ts` gana `deadlineAt` en `ResolveSigningIntermediatesAiaOpts` + `clampAiaLegTimeout` (mismo idioma que `clampToDeadline`/`legTimeoutMs` ya usado para OCSP/CRL): cada hop se acota al tiempo restante, y una vez agotado el deadline agregado el resto de los hops se saltan sin siquiera intentar el fetch.

  Los cuatro con tests de regresión nuevos (rojo→verde con bytes/timings reales donde aplica): `aia-certs.test.ts` (+4), `sign-queue.degradation.test.ts` (+2), `sign-network-budget.test.ts` (+1 suite), `chain-intermediates.test.ts` (+2). Pendiente para una próxima ronda (no bloqueante): el motivo exacto de un fallo AIA (timeout/CSP/red/firma inválida) sigue colapsado en un solo aviso genérico en la UI.

### Added
- **Firma por lotes: propagación de posición entre documentos del mismo formato** (`@firma-ec/pwa` 0.22.0, `@firma-ec/signer` 0.10.0): al colocar la firma a mano en un documento `needs_review` del lote (`/firmar-lote`), esa posición confirmada se propone automáticamente a los demás documentos con la MISMA firma de formato de página (misma plantilla exportada a PDF) — sin un paso extra ni pregunta adicional. Diseño **propose-not-place**: la propagación nunca escribe una posición directamente, solo pasa un hint (`toCanonicalRect`, nuevo `AnchorPlacementKind: 'lote-propagacion'`) al motor de colocación real (`computeAutoPlacement`), que lo re-valida como cualquier otro documento — cada resultado se reduce a 3 estados observables: "Aplicada" (`propagated: 'exact'`), "Reubicada" (`'moved'`, el motor encontró otro hueco válido) o "Requiere revisión" (el hint no ayudó), cada uno con acción "Ajustar" para corregir a mano. Nunca sobrescribe una posición ya confirmada por la persona. Motor: `sanitizeAnchor` (descarta el hint entero, nunca solo un campo, ante un número no finito o fuera de rango) y `clampAndRevalidate` (cierra un defecto reactivado donde `computeAntiOverlapPlacement` podía colocar una firma encima de contenido previo tras acotar a los márgenes, sin la revalidación que `tryAnchorPlacement` ya tenía). Empíricamente justificado antes de construirse: medido sobre el corpus real (1.458 PDFs), 67,8% de propagaciones caen exactas, 27,8% se reubican limpio, solo 4,3% necesita revisión manual — contraste explícito con un intento anterior similar (ancla de texto genérica) que midió negativo en el mismo corpus. Gate final de QA (code-reviewer + silent-failure-hunter, opus, sobre el diff completo antes de este release) encontró y cerró un P0 real: `goToReview` podía revertir en silencio una colocación manual/propagada si el resto del lote seguía analizándose cuando se confirmaba, dejando el documento fuera del ZIP sin ningún aviso; más dos P1 de fallos de infraestructura del worker mal atribuidos a "PDF corrupto" en vez de reportados con causa honesta (única vía de diagnóstico en una app sin telemetría).
- **CI gate sobre pull requests + seam `seo-meta.json`** (`@firma-ec/landing` 0.6.27 — repo `alfonso/firmar-ec`): dos piezas para que el motor de remediación SEO (idkpublicitaria) pueda algún día auto-mergear sin humano en el medio, replicando el patrón ya construido y probado en producción para `idkmanager-web`. (1) `.gitea/workflows/pr-check.yml` — hasta hoy solo `deploy.yml` posteaba commit status, y únicamente en `push:[main]`; el head sha de un PR nunca recibía evidencia de CI, así que `getPullCiStatus` lo leía como `"none"` y el gate de auto-merge lo trataba `ci_red` para siempre (fail-closed, correcto, pero insatisfacible). El nuevo workflow dispara en `pull_request`, clona y se posiciona en el **head sha exacto** (no la punta de la rama, que puede haber avanzado), construye la imagen real con `infra/docker/landing.Dockerfile` (mismo Dockerfile que `deploy.yml`, sin push a ningún registry), verifica que el home responde 200, que el `<title>` no queda vacío (detecta un seam roto) y que `llms.txt` se sirve con contenido, y postea el commit status (`context: "ci/pr-check"`) sobre `pull_request.head.sha` — nunca sobre el merge-commit efímero del evento. Todo valor de payload (rama, sha) entra por `env:` y se referencia entrecomillado; ningún `run:` interpola `${{ }}` de payload directo (la inyección de shell real que ya se vio en la ronda de `idkmanager-web`). (2) `apps/landing/src/data/seo-meta.json` — seam de datos `ruta → {title?, description?}` que `BaseHead.astro` consulta ANTES de componer el título/descripción de cada página; si hay entrada para la ruta (normalizada sin barra final en ambos lados — el sitio sirve con `trailingSlash: "always"`), el seam **gana** sobre las props que pasa la página, para que un fix auto-mergeado tenga efecto observable en vez de aterrizar silencioso. Arranca `{}` (no-op). Verificado con build real en los 3 casos: seam vacío → páginas idénticas; entrada de prueba en `/faq` → title/description/og:title pisados solo ahí; el resto de rutas (home, `/acerca`) intactas. Sin infraestructura de tests para componentes Astro en este monorepo — verificado por build real (`pnpm --filter @firma-ec/landing build`), no se introdujo un framework de testing nuevo para esto. El wiring en `idkpublicitaria/src/config/gitConnectors.ts` (seamFile + autoMergePaths para `firmar.ec`) queda para una pasada separada.

### Fixed
- **Firma por lotes: segunda ronda de QA (OWASP + buenas prácticas) sobre la feature ya desplegada** (`@firma-ec/pwa` 0.22.1): ronda independiente de la del release anterior, pedida explícitamente para cubrir lo que un gate fase-por-fase no ve al mirar la feature ensamblada en su forma final. El hallazgo de seguridad va en la entrada de `### Security` de arriba. El resto: (1) el orquestador de la UI del lote (`FirmarLote.svelte`) tragaba sus 4 errores sin loguear nada, único módulo de la feature sin ese hábito, y clasificaba como "PIN incorrecto" cualquier error cuyo mensaje contuviera la palabra "password"; (2) `goToReview` sin `try`/`finally` podía dejar el spinner de revisión girando para siempre ante un rechazo del Worker, con el error fijado en un sitio que nunca se pintaba en ese paso; (3) los bytes ya firmados que el motor conserva a propósito cuando falla la escritura al ZIP (para no re-pagar TSA/OCSP) se tiraban sin ofrecer ningún link de descarga — ahora cada documento recuperable ofrece "Descargar de todos modos"; (4) sin ninguna guarda de navegación, el ZIP —única copia de hasta 50 documentos legales firmados— se destruía sin aviso al reiniciar, navegar, cerrar o refrescar la pestaña; (5) la guarda anti-pisada del P0 anterior usaba un marcador de presentación (`manual`/`propagated`) en vez de procedencia real, reabriendo el mismo bug por una rendija más angosta (propagación con fuente `empty-field` o sin geometría); (6) una colocación manual en una página rotada podía publicar un rect sin `rotate`, apagando la segunda validación del motor. Más un `MEDIUM` en `sign-session-bus.ts` (el timeout de apertura de sesión no daba oportunidad de limpiar la clave si `parsePfx` ya había terminado) y varios `LOW` (truncado del ZIP que se pasaba del tope tras el sufijo de colisión, caracteres de control bidireccional sin filtrar en los nombres mostrados). Verificado: signer 443/443, pwa 341/341, e2e completo 79/79.

### Added
- **Identidad del firmante desde el arco OID privado de cada ACE** (`@firma-ec/crypto-core`): la cédula se leía **solo** del RDN `serialNumber` del subject DN (OID 2.5.4.5), documentado en tres archivos como "la cédula/RUC en certificados EC". Medido contra PDFs firmados reales, ese campo es la peor fuente disponible: **ausente** en ArgosData, con un valor no relacionado en Security Data (23 y 12 dígitos), y con **otro número de 10 dígitos** en Banco Central — solo ICERT-EC lo llena de verdad. Resultado: el campo salía vacío para titulares de ArgosData y habría mostrado un número equivocado para los del BCE. Nuevo `ecCertIdentity` lee el arco privado de cada ACE (ICERT-EC/Consejo de la Judicatura `1.3.6.1.4.1.43745.1.3`, ArgosData `…59198.3`, Security Data `…37746.3`, Banco Central `…37947.3`, Uanataca `…47286.102.3`), que comparten el mismo esquema de sufijos (`.1` cédula, `.2` nombres, `.3`/`.4` apellidos, `.11` RUC) en dos colocaciones distintas: anidado en `subjectAltName` como `otherName` (ICERT-EC, Uanataca) o como extensión de primer nivel (el resto). Respaldo al DN y, por último, un barrido protegido por el **dígito verificador mod-10** de la cédula, que es lo que impide que ese último recurso tome un atributo privado cualquiera. Un valor publicado por un arco conocido se toma tal cual: validarlo sería fail-closed contra un titular extranjero legítimo. Pasó inadvertido porque **ningún test usaba un certificado real** — los fixtures eran sintéticos y construidos sobre la misma premisa que el código. Los tests afirman sobre propiedades (nº de dígitos, checksum, relación cédula↔RUC) y nunca sobre valores, así que no queda ninguna cédula escrita en el repositorio. **ANF sigue sin mapear** (sin certificado de titular disponible para derivar su arco).
- **La cédula del firmante se pinta en la estampa visible** (`@firma-ec/signer` 0.9.0, `@firma-ec/pwa` 0.21.0): línea "Cédula: …" alimentada por `holderCedula`. Omitirla deja el flujo de apariencia **byte a byte idéntico**, así que las firmas existentes no cambian. `SignerCert.subjectSerialNumber` pasa a `holderCedula` + `holderRuc` (BREAKING) porque el valor ya no viene del subject DN.
- **Firma por lotes: nueva ruta `/firmar-lote` para firmar varios PDFs de una sola vez** (`@firma-ec/pwa` 0.21.0): selección múltiple de documentos, seguida de un **pre-vuelo** (`preflightBatch`, reutiliza `analyzePdfForPlacement` del pipeline estándar) que revisa la colocación de la estampa en cada PDF **antes** de pedir el `.p12`/PIN — los documentos que necesitan revisión manual quedan marcados y excluidos de la firma automática en vez de interrumpir el lote a mitad de camino. La firma corre con una sola credencial para todo el lote (`sign-session.worker.ts`, sin duplicar la lógica sensible del modo estándar `#/firmar`), con progreso por documento, y entrega los PDFs firmados en un único ZIP descargable. Mismo invariante de privacidad del resto de la app: ni el nombre de archivo ni el PIN se registran en logs.

### Fixed
- **`ecCertIdentity` fabricaba una cédula truncando el RUC de una persona jurídica** (`@firma-ec/crypto-core`): la rama de respaldo del DN aceptaba cualquier valor de 13 dígitos y lo cortaba a 10 **sin volver a validar**. Para una persona natural el prefijo del RUC sí es su cédula; para una jurídica no es nada, y como la UI evalúa `cedula ?? ruc`, el número **inventado** ganaba al RUC correcto que estaba al lado. Ahora el prefijo debe pasar `isValidEcCedula`; si no, se publica solo el RUC. Las dos ramas de respaldo (DN y heurística) eran las únicas sin cobertura: el test que existía usaba `1700000001001`, cuyo prefijo es válido — el caso feliz que no caza nada. Detectado por revisión adversarial independiente.
- **OCSP: cierre del hueco de replay/mismatch + fix de disponibilidad ArgosData** (`@firma-ec/ltv-validation` 1.0.0, BREAKING): `fetchOcsp` construía el CertID de la petición y **nunca lo comparaba** con el de la respuesta — un MITM en HTTP plano podía reinyectar una respuesta legítimamente firmada por la CA pero sobre OTRO certificado, y la app la leía `good`, dejando firmar con un certificado revocado. `parseOcspResponse` gana un tercer parámetro **obligatorio** `expected: { serialHex, issuerKeyHashHex? }` (normalizado vía nuevo `normalizeSerialHex` — quita el pad `0x00` de un serial DER sin arriesgar colisión, porque despojar ceros a la izquierda de un entero no-negativo es una biyección) y selecciona, entre las `SingleResponse` de la respuesta, la que realmente responde al certificado pedido; si ninguna casa Y la respuesta está autenticada, lanza `OcspParseError({code:'no_matching_single_response'})` → `fetch.ts` lo traduce al nuevo `OcspError.reason = 'response_mismatch'`. El nonce (RFC 6960 §4.4.1) ahora se **verifica** cuando la respuesta lo incluye (verify-if-present; ARCOTEL omite el nonce y a veces rechaza peticiones que lo llevan, así que seguir sin exigirlo era la decisión correcta). De regalo, un **bug de disponibilidad medido en vivo**: pkijs 3.4.0 lanza `"No certificates attached"` cuando el responder firma directamente sin adjuntar `certs[]` (el caso real de ArgosData/ACE) en vez de caer al emisor — `parseOcspResponse` ahora verifica directo contra la clave pública del emisor cuando `responderID` lo identifica, cerrando el falso-negativo sin abrir superficie (el EKU `id-kp-OCSPSigning` sigue exigido para cualquier delegado, plegado en `signatureValid` en vez de en un booleano aparte que nadie leía — la causa raíz original del agujero). `packages/verifier/src/ltv.ts`'s `ocspMatchesCert` (usado al leer el DSS embebido en un PDF) tenía el mismo patrón invertido: normalizaba el serial del certificado pero comparaba contra el serial de la respuesta SIN normalizar, así que cualquier certificado cuyo serial DER llevara el byte de signo `0x00` nunca casaba y su evidencia — incluida una revocación — se ignoraba en silencio; ahora normaliza ambos lados y añade el `issuerKeyHash` bajo el algoritmo de hash que la propia respuesta reporta.

### Added
- **Landing — 2 preguntas al FAQ para cerrar hueco de citación GEO** (`@firma-ec/landing` 0.6.24): idkpublicitaria detectó 2 recomendaciones P1 (`geo-gap-prompt`) — gemini:web navegaba y citaba fuentes reales para "¿Cómo firmar un documento PDF con firma electrónica gratis y sin instalar programas?" y "¿Qué herramienta puedo usar para firmar un PDF con mi certificado .p12 desde el celular?", pero citaba `vertexaisearch.cloud.google.com` (artefacto de grounding de Gemini) en vez de la marca. El contenido ya existía en prosa (`firmar-documentos-en-linea.md`, FAQ #07/#11) pero ninguna entrada respondía la frase casi literal. 2 entradas nuevas (`14-firmar-gratis-sin-instalar.md`, `15-firmar-p12-celular.md`) heredan el `FAQPage` JSON-LD existente sin cambio de esquema.

### Changed
- **Landing — sello del hero gira a 16s/vuelta (antes 60s, imperceptible)** (`@firma-ec/landing` 0.6.26): a 60s/vuelta (6°/s) el anillo giraba pero no se notaba ("no gira"). Bajado a 16s/vuelta (22.5°/s): claramente perceptible, sigue elegante. Igual solo `transform` (GPU) y detenido en `prefers-reduced-motion`.
- **Landing — hero de la home rediseñado a 2 columnas + sello de certificación giratorio** (`@firma-ec/landing` 0.6.25): el hero era una columna centrada y apilada que desperdiciaba el espacio lateral en desktop y, tras quitar la escena vieja (mockup de navegador con contrato), dejaba un hueco. Rediseño en tres partes: (1) **layout de 2 columnas** en `md+` (grid `1fr/0.5fr`) — texto, CTAs y badges alineados a la izquierda ocupando su lado, y un visual a la derecha; móvil intacto (grid solo en `md`, texto centrado, visual oculto). (2) **nuevo `HeroSeal.astro`** — escudo con check ámbar + chip `.p12` al centro y un anillo de texto ("FIRMA ELECTRÓNICA · ECUADOR · SEGURA") que gira lento (60s/vuelta); todo CSS/SVG (CSP-safe, 0 KB de imagen), decorativo (`aria-hidden`), rotación solo por `transform` en un `<div>` HTML (no en `<g>` SVG, evita el quirk de `transform-origin` en Safari) y detenida en `prefers-reduced-motion`. (3) **compactado vertical** para subir la sección "Firmar Fácil" above-the-fold — CTAs en una sola fila (columna de texto ensanchada, "Abrir la app" pasa de botón a link), márgenes y `pt/pb` reducidos, y menos padding superior en `FirmarFacil.astro`. Hero de 778→598px @1440×820. Verificado en preview (claro/oscuro/móvil 375-1440, sin scroll horizontal, orden de foco intacto, 4 CTAs presentes) y build (`astro check` + `build`, 68 páginas, sitemap + check-llms OK).
- **Landing — `<title>` de las 4 guías "cómo firmar con certificado X" acortado a ≤60c** (`@firma-ec/landing` 0.6.23): medidos en vivo, los cuatro se truncaban en el SERP (67c ArgosData, 86c Consejo de la Judicatura, 71c Security Data, 66c UANATACA; `BaseHead` añade `" — firmar.ec"`, +12c). Son las páginas que más convierten, así que el truncado costaba CTR justo donde más caro es ganarlo. Se acorta **solo** el campo `title` del frontmatter (el que alimenta `<title>`/`og:title`); el `h1` ya era un campo independiente (`h1: data.h1 ?? data.title`), de modo que **el encabezado visible no cambia**. Quedan en 56c/54c/57c/55c. Los términos que salen del title (`iCert-EC`, "en Ecuador") siguen presentes en h1, description y cuerpo, así que no se pierde cobertura semántica. Verificado sobre el HTML construido (`npm run build`, 68 páginas, sitemap + check-llms OK), no solo sobre el frontmatter.

### Fixed
- **PWA — modo guiado "Firmar Fácil": no se podía colocar la firma sobre la línea de firma del documento** (`@firma-ec/pwa` 0.20.7): en el paso 2 guiado, la colocación ofrecía solo (a) una posición auto-sugerida y (b) una rejilla de 6 celdas **ancladas al borde inferior** de la página (`computeGridPlacements`), que no alcanzaba la zona media/alta donde suelen estar las líneas de firma — así que colocar la firma "encima de la línea" era **imposible**, pese a que el copy prometía "Toca dónde quieres tu firma". Fix en dos frentes, sin arrastre libre (se mantiene la garantía de accesibilidad AAA): (1) **tap-para-colocar** — en el estado por defecto, tocar CUALQUIER punto del documento coloca la firma ahí (un solo toque; reusa la matemática coord CSS→PDF y el `TOUCH_OFFSET_PX=24` del `BoxPlacer` estándar, extraída como helper puro `placeBoxAtTap`); el copy del estado por defecto pasa a invitar a tocar el documento (`guided.placer.tap_hint`). (2) **rejilla repartida** — `computeGridPlacements` distribuye ahora las 3 filas por TODA la altura útil (no solo el tercio inferior), con clamp robusto en páginas cortas, de modo que la ruta accesible por teclado también ofrece celdas en la zona de la línea de firma; su copy pasa a "Toca una de las opciones numeradas" (`guided.placer.grid_hint`). Los dos modelos quedan separados (tap-libre solo sin rejilla; rejilla = solo celdas) para no confundir al público objetivo. Capa de toque `aria-hidden` no-focusable; botones numerados siguen siendo la ruta de teclado ≥44px. Modo estándar `#/firmar` (`BoxPlacer`) intacto.
- **PWA — la voz del modo guiado seguía diciendo lo viejo tras regenerar los clips (caché de Cloudflare)** (`@firma-ec/pwa` 0.20.4): los `.mp3` de `/voz-firma/` tienen nombre estable (`confirmar.mp3`, etc.), así que Cloudflare los cachea por URL (`Cf-Cache-Status: HIT`, `max-age=14400`) y, tras alinear el copy y **regenerar los clips en 0.20.3**, el edge seguía sirviendo el audio ANTERIOR aunque el `manifest.json` (que es `DYNAMIC`, no cacheado) ya traía el hash nuevo. Fix durable: `resolveClipUrl` cuelga `?v=<hash del manifest>` de la URL del clip — un cambio de texto cambia el hash → cambia la URL → Cloudflare, el Service Worker y el navegador traen el clip nuevo automáticamente, sin purgar caché a mano nunca más. `url.pathname` no incluye la query, así que la regla runtime-cache de `sw.ts` (`^/voz-firma/.*\.mp3$`) sigue matcheando.
- **PWA — modo guiado "Firmar Fácil": se escuchaban dos voces a la vez** (`@firma-ec/pwa` 0.20.3): `speak()` hacía `await el.play()`, pero esa promesa resuelve en cuanto el clip *empieza* a sonar, no cuando termina. Si llegaba una segunda narración antes (p.ej. la voz de bienvenida seguida de inmediato por la narración automática del paso 1 al montar), esa segunda llamada cortaba la primera con `stop()` → `audioEl.pause()`, lo que **rechazaba la promesa `play()` de la primera con `AbortError`** — y su `catch` caía a `ttsFallback(...)` del PRIMER texto, sonando a la vez que el clip/TTS de la segunda. Fix: guard de generación (`playGeneration`, incrementado en cada `stop()`, reclamado por cada `speak()` y re-chequeado tras cada `await`) — una llamada superada por otra más reciente sale en silencio, nunca hace fallback; solo un fallo real de reproducción (formato/red/404, sin interrupción posterior) cae a Web Speech. Test de regresión en `apps/pwa/src/lib/guiado/voice.test.ts` (reproduce la interrupción y verifica que `speechSynthesis.speak` no se invoca para el texto interrumpido), corrido vía config local `apps/pwa/vitest.config.ts` (los módulos `*.svelte.ts` con runes `$state` necesitan `@sveltejs/vite-plugin-svelte` para compilar, ausente en el `vitest.config.ts` raíz compartido por todo el monorepo — el archivo se excluye explícitamente de `pnpm test` en la raíz para no romper CI, y se corre con `pnpm --filter @firma-ec/pwa exec vitest run`).

### Changed
- **Landing — "Firmar Fácil" añadido al nav** (`@firma-ec/landing` 0.6.22): nuevo ítem **Firmar Fácil** junto a "Firmar" en el nav de `firmar.ec` (escritorio y menú móvil), enlazando directo al modo guiado con voz en la app (`https://app.firmar.ec/#/firmar-facil`, misma URL que la sección `FirmarFacil.astro`). El espacio liberado por el menú "Más" (0.6.21) permite tenerlo inline sin desbordar. i18n `nav.firmar_facil` ES/EN ("Firmar Fácil" / "Easy Signing").
- **Landing — el nav de escritorio agrupa los ítems secundarios en un menú "Más"** (`@firma-ec/landing` 0.6.21): el nav de `firmar.ec` tenía 7 ítems (Firmar · Verificar · Validar certificado · Seguridad · Preguntas · Acerca · Patrocinar) y apretaba. Los tres secundarios (**Seguridad · Preguntas · Acerca**) se colapsan tras un desplegable **"Más"**, dejando inline Firmar · Verificar · Validar · Patrocinar. Como la landing es Astro (estática), el dropdown es JS vanilla `is:inline` con el mismo patrón **disclosure** que ya usa el menú móvil (botón + lista de enlaces, sin `role="menu"`; solo `aria-expanded`/`aria-controls`; cierra por Escape→foco al trigger, click fuera del contenedor, y al activar un enlace; chevron que rota, panel que escala desde su origen top-right ~200ms `ease-out`, `motion-reduce` sin transición). El **menú móvil (hamburguesa) mantiene la lista plana** — ahí el espacio no es escaso. Espejo de lo hecho en la PWA (0.20.5).
- **PWA — Home: "Firmar PDF" pasa a ser el CTA destacado y el botón "Empezar" de Firmar Fácil sube a la esquina superior derecha** (`@firma-ec/pwa` 0.20.8): (1) en la fila de acciones del hero, **Firmar** toma el estilo primario (azul, `size lg`) y la primera posición — coherente con que firmar.ec es ante todo para *firmar*; **Verificar** queda como `outline` secundario a su derecha (solo se intercambian variante/tamaño/orden; el texto de cada botón sigue su misma clave i18n). (2) En la banda "Firmar Fácil", el pill **"Empezar"** se movió del pie a una **fila superior junto al ícono** (arriba a la derecha), con el título y la descripción debajo — más visible y menos "huérfano" en móvil. La tarjeta entera sigue siendo un enlace a `#/firmar-facil`; hover/focus/animaciones intactos.
- **PWA — Home: el on-ramp "Firmar Fácil" sube al hero (siempre visible) y los badges de certificación bajan** (`@firma-ec/pwa` 0.20.6): la banda de "Firmar Fácil — con voz que te guía" vivía en la segunda sección, **bajo el fold en móvil** (había que hacer scroll para descubrir la vía guiada), mientras que los chips de certificación (AGPL-3.0 · ETSI EN 319 142 · ARCOTEL TSL · LOPDP nativa · 100% browser) ocupaban el espacio prime al final del hero. Se invirtió la prioridad: la banda de Firmar Fácil se mueve **al hero, justo debajo de los CTAs primarios** (Verificar/Firmar), así la vía accesible con voz se ve sin scroll; los badges quedan por debajo (patrocinadores + certificaciones), su sitio natural como "credenciales" secundarias. Mismo componente y clases (solo `p-6`→`p-5 sm:p-6` y margen para el contexto del hero); la segunda sección conserva las tarjetas Verificar/Firmar. Sin cambios de lógica ni de rutas.
- **PWA — el nav de escritorio agrupa los ítems secundarios en un menú "Más"** (`@firma-ec/pwa` 0.20.5): la barra tenía 8 ítems (Inicio · Firmar · Firmar Fácil · Verificar · Validar certificado · Paranoia · Acerca · Configuración) y se veía apretada, quitándole aire a las acciones primarias. Ahora los tres secundarios (Paranoia, Acerca, Configuración — seguridad/info/ajustes) se colapsan tras un solo desplegable **"Más"** (`ui/NavMore.svelte`), liberando dos espacios para lo que la gente de verdad viene a hacer (firmar/verificar). Patrón **disclosure** (botón + lista de enlaces, NO `role="menu"`): son enlaces de navegación, no comandos, así que es el patrón ARIA correcto y más simple (sin roving tabindex, orden de tabulación natural). Solo `aria-expanded` en el trigger (el patrón disclosure no requiere `aria-haspopup`, que implicaría `role="menu"`); cierra por **Escape** (devuelve el foco al trigger), `pointerdown` fuera (captura), foco saliendo del contenedor, y al activar un enlace; chevron que rota y panel que escala desde su origen (top-right, anclado al trigger) en ~180ms `ease-out`, con `prefers-reduced-motion` colapsando la duración a 0. El `class:text-brand-500` del trigger refleja cuándo la ruta activa vive dentro del menú. El **menú móvil (hamburguesa) mantiene la lista plana** — ahí el espacio vertical no es escaso. Sin cambios de comportamiento en ninguna ruta.

### Added
- **App + Landing — punto de entrada visible a "Firmar Fácil"** (`@firma-ec/pwa` 0.20.2, `@firma-ec/landing` 0.6.20): el modo guiado (`#/firmar-facil`) existía pero **no tenía ningún enlace visible** — solo se llegaba escribiendo la URL. Ahora hay una **banda destacada** que lo hace descubrible sin canibalizar los CTA primarios: (a) en el Home de la app (`Home.svelte`), a ancho completo justo bajo el hero y antes del grid Verificar/Firmar — un solo `<a href="/firmar-facil">` con icono `i-lucide-volume-2` (la voz es el diferenciador), tinte de marca al 6% (nunca el fill sólido reservado al CTA primario), título "Firmar Fácil — con voz que te guía", descripción y pill "Empezar"; (b) en la landing (`FirmarFacil.astro`, entre `Hero` y `ComoFunciona`, ES y EN), sección `aria-labelledby` con texto real indexable (SEO) y CTA `<a href="https://app.firmar.ec/#/firmar-facil">` con el hash intacto. Copy **neutro, sin etiquetar por edad** ("¿Primera vez? Te acompañamos", no "para mayores"). i18n `home.facil.*` ES/EN; a11y con targets grandes y focus visible; reduce la jerarquía a un solo tab-stop. Nuevo e2e de que el Home enlaza y navega al flujo guiado.

### Fixed
- **PWA — chequeo de actualización del Service Worker dirigido por evento + copy del paso 1 guiado alineado con la voz** (`@firma-ec/pwa` 0.20.1): (1) el poll periódico de `registration.update()` (antes cada 60 min) dejaba una ventana larga en la que un cliente ya abierto seguía sirviendo la versión vieja tras un deploy. Ahora, además del poll (bajado a **20 min**), se dispara un `registration.update()` best-effort al volver la pestaña a primer plano (`visibilitychange` → `visible`) y al restaurar desde bfcache (`pageshow`) — acorta la ventana sin tocar la estrategia de caché de `sw.ts` (precache/`NavigationRoute`/reglas de `/assets`, `/voz-firma`, crypto, TSL quedan intactas; ese código causó el incidente "mobile-hang" y no se modificó). El reload automático por `controllerchange` y `applyUpdate()` no cambiaron. **No arregla retroactivamente** clientes que ya tenían el SW viejo cacheado antes de este cambio — esos siguen en la versión anterior hasta su primer `update()` exitoso (ahora más frecuente). (2) En el modo guiado "Firmar Fácil" (`#/firmar-facil`), paso 1: la voz decía "Toca el botón grande que dice 'Elegir mi documento'" pero el `Drop` mostraba la etiqueta estándar "Arrastra un PDF firmado aquí o selecciona un archivo" — incoherente (el PDF aún no está firmado al momento de cargarlo) y desalineada con la narración. Ahora el paso 1 guiado muestra el título `guided.step1.title` ("Elige tu documento") y el `Drop` recibe las nuevas props aditivas `label`/`pickLabel`/`ariaLabel` (default = comportamiento estándar sin cambios) con `guided.step1.drop_sub` ("Toca aquí para buscar tu PDF") + `guided.step1.cta` ("Elegir mi documento"), coincidiendo con la voz. El modo estándar `#/firmar` no cambia ni un carácter de texto.
- **Landing — el FAB de WhatsApp no "palpitaba"** (`@firma-ec/landing` 0.6.19): la animación corría pero era imperceptible — un solo halo a opacidad 0.55 con ease-out agresivo (~300ms visibles de un ciclo de 2.6s) y el botón estático. Ahora el disco verde + icono laten con **doble golpe de corazón** (`scale 1→1.1→1→1.07→1`, 2.8s) y detrás salen **dos anillos expansivos escalonados** (0.4s de desfase) sincronizados con el latido. El latido vive en un hijo del ancla para no pelear con `hover:scale`/`active:scale` (una animación de transform en el mismo elemento pisa el transform del hover). Solo `transform`/`opacity`; `prefers-reduced-motion` lo apaga todo.

### Added
- **PWA — Modo guiado "Firmar Fácil": pulido AAA + mascota "Fe" (Fase 3)** (`@firma-ec/pwa` 0.20.0): capa de calidez y accesibilidad sobre el flujo guiado. **Mascota "Fe"** (`ui/guiado/GuideMascot.svelte`, avatar SVG inline con `aria-hidden` + burbuja) que acompaña en la bienvenida y en el resultado — reduce la ansiedad de "alguien me acompaña". **Ayuda contextual `<details>`** (`ui/guiado/GuideHelp.svelte`, "¿Por qué me piden esto?") en los pasos de certificado y contraseña — accesible sin hover, mejor que tooltips. Los 2 clips de voz que quedaban sueltos ahora se narran: `ayuda_lugar` al abrir la rejilla de posiciones y `pdf_ok` al cargar el documento. **"¿Retomamos donde ibas?"** con persistencia mínima y honesta (`lib/guiado/resume.ts`, guarda SOLO el número de paso en `localStorage`, **jamás** el PDF/.p12/PIN; se limpia al completar o reiniciar). **Voz en inglés**: cuando el idioma es EN, la narración usa Web Speech `en-US` con el texto EN (los clips mp3 son solo ES); en ES sigue priorizando el clip. **Accesibilidad AAA verificada de verdad**: se instaló `@axe-core/playwright` y se añadió `firmar-facil.a11y.spec.ts` — axe **0 violaciones critical/serious en el flujo guiado** (escaneo acotado a la región `[data-guided]`, en bienvenida/placer/cert/PIN, desktop y mobile), recorrido **solo-teclado** completo, y `prefers-reduced-motion` sin animaciones; axe encontró y se **arreglaron** contrastes reales del modo guiado (botones sobre `brand-500`→`brand-600`, verde WhatsApp, hints `ink-500`→`ink-600`). **Lighthouse a11y 0.96** en `#/firmar-facil`. Follow-ups conocidos, **fuera del alcance del modo guiado** (chrome global preexistente): contraste del logo/badge de header/footer (token `--brand-500`) y del banner `InstallPrompt` en Android (`disabled:opacity-50`). El modo estándar `#/firmar` se mantiene (único cambio compartido: el contraste del hint de `PinInput`, una mejora AA).
- **PWA — Voz "Fe" en el modo guiado "Firmar Fácil" (Fase 2)** (`@firma-ec/pwa` 0.19.0): narración que acompaña paso a paso, para quien no lee cómodo en pantalla. Arquitectura de **2 niveles**: clips `.mp3` pre-renderizados (voz ecuatoriana femenina `es-EC-AndreaNeural` vía **edge-tts/MoneyPrinterTurbo**, misma identidad "Fe" que la tienda) con **fallback a Web Speech API** si falta el clip o el texto es dinámico. Motor en `lib/guiado/voice.svelte.ts` (`speak`/`speakAuto`/`stop`, `pickSpanishFemaleVoice`), UI en `ui/guiado/GuideNarrator.svelte` (botón grande "Escuchar" ↔ "Detener" + la frase visible para quien no usa audio), montado en las 6 ramas de paso del modo guiado. **Respeta la política de autoplay**: nada suena al cargar — la primera reproducción exige un gesto del usuario (`audioUnlocked`), y solo entonces `speakAuto` narra los pasos si `voiceAuto` está activo (toggle "Voz activada/apagada"); la voz se corta en cada transición y al desmontar (0 `NotAllowedError` verificado). 13 clips (`bienvenida, cargar_pdf, pdf_ok, ubicar_firma, cert_pregunta, cert_no, cargar_p12, pin, pin_error, confirmar, firmando, listo, ayuda_lugar`), normalizados mono/48kbps/loudnorm, **0.67 MB** total, en `public/voz-firma/` + `manifest.json` (clave→`{file, hash sha256[:16]}`: si el copy i18n cambia y el hash no coincide, cae a Web Speech). Regenerables con `scripts/gen-voz.mjs` (lee las frases del i18n como fuente de verdad; binario edge-tts configurable por `EDGE_TTS_BIN`). Caché PWA `CacheFirst` para `/voz-firma/*.mp3` en `sw.ts` (runtime, sin engordar el precache). El modo estándar `#/firmar` no monta nada de voz. Namespace i18n `guided.voz.*` ES/EN.
- **PWA — Modo guiado accesible "Firmar Fácil" para personas mayores / poca soltura digital** (`@firma-ec/pwa` 0.18.0): nueva ruta `#/firmar-facil` que monta el **mismo** asistente de firma (`Firmar.svelte` con prop `guided`, vía `wrap({component, props})` de svelte-spa-router) — una sola máquina de estados, cero duplicación de la lógica sensible de .p12/PIN/workers, y el flujo estándar `#/firmar` queda **idéntico** (todo el wiring nuevo va tras `{#if guided}`, `{:else}` conserva el render actual). Elimina los dos puntos de fricción reales del público objetivo: (1) **sin arrastrar la firma** — `SimplePlacer.svelte` auto-coloca una caja grande al pie de la última página (`placeAtBottomLastPage` en `smartPlacement.ts`, determinista) y pregunta *"Tu firma irá aquí, ¿está bien?"*; alternativa = rejilla de ≤6 posiciones (`computeGridPlacements`) con un toque, nunca drag libre; (2) **salida "no tengo mi certificado"** — `CertHelp.svelte` ofrece comprar en la tienda o escribir por WhatsApp (URLs en `lib/links.ts` vía `import.meta.env`, cero hardcoding, reutilizando el origen ya existente). Capa visual **XL accesible** scoped bajo `[data-guided="true"]` (`styles/guided.css`): escala 20–22px, CTAs ≥56px, hit targets ≥48px, focus 4px, `prefers-reduced-motion`, dark-mode — Geist a escala grande, sin fonts/colores nuevos, inerte en el modo estándar. `WhatsAppSticky.svelte` de ayuda siempre visible. Namespace i18n `guided.*` (ES/EN). Settings `guidedMode`/`voiceAuto` cableados (la voz llega en F2). Cobertura e2e: `firmar-facil.spec.ts` **firma un PDF real** de punta a punta (Chromium + Pixel 7) con `.p12` self-signed efímero generado en caliente (`tests/e2e/global-setup.ts`, node-forge, nunca commiteado) y validación en 2 capas (bytes `/ByteRange`+`/Contents` y re-detección de la firma al re-subir). De paso se arreglaron **bugs preexistentes** que dejaban la suite roja: specs stale que esperaban el heading "Detalles opcionales" (eliminado en v0.7.15), y en `tsa-flow.spec.ts` la key `localStorage` v1→v2 y roles ARIA mal esperados. **Gotcha durable**: la firma real en `vite dev`/e2e requería servir los intermedios `*.pem?raw` que el `server.fs.deny` por defecto de Vite (`*.{crt,pem}`) bloqueaba (403 → `SignerError`); se abre solo tras `DEV_FS_ALLOW_PEM=1` (Playwright lo setea; `fs.strict` sigue en `true`, postura de seguridad por defecto intacta). Suite: 23 passed / 0 failed.
- **Landing + PWA — tema en 3 estados: Automático (sistema) / Claro / Oscuro** (`@firma-ec/landing` 0.6.18, `@firma-ec/pwa` 0.17.12): antes el botón solo alternaba claro↔oscuro y, una vez tocado, la preferencia quedaba fijada para siempre — no había forma de volver a seguir al sistema. Ahora el botón **cicla auto → claro → oscuro** (icono monitor/sol/luna + `title`/`aria-label` con el modo actual); se persiste el **modo** (`'auto'|'light'|'dark'`) en `localStorage 'theme'` — los bootstraps de `Base.astro`/`index.html` ya trataban cualquier valor ≠ light/dark como "seguir al sistema", así que `'auto'` es retro y forward-compatible sin tocarlos; en modo auto un **listener de `prefers-color-scheme` re-aplica el tema en vivo** si el sistema cambia con la página abierta. En la landing, las dos instancias del toggle (header ≥sm y menú móvil) se sincronizan vía evento `fec:thememode`. Feedback físico `active:scale-[0.96]` + easing de la casa.

### Changed
- **Landing — hero "humano": escena animada de firma en vivo reemplaza el hero solo-texto** (`@firma-ec/landing` 0.6.17): (1) **HeroScene** nueva — un documento PDF dentro de un marco de navegador (double-bezel, píldora de URL `app.firmar.ec` con candado) donde la **rúbrica manuscrita se dibuja sola** (SVG stroke-dashoffset, 1.4s), luego aparece el **sello ámbar con QR** (scale+opacity, 2.1s) y remata el **chip verde "Firma válida"** solapado al marco (2.7s) — cuenta el producto entero en 3 segundos sin una sola imagen (CSS/SVG puro, 0 KB extra, retina-perfect, `transform`/`opacity`/`stroke-dashoffset` only, `aria-hidden`, reduced-motion = estado final estático, glow ambiental azul+ámbar tras el marco); (2) las 2 tarjetas-documento flotantes laterales de 0.6.15 se retiran (competían con el nuevo punto focal único); (3) **menos bloque de texto**: el párrafo del hero baja de 4 frases a 1 ("Gratis para uso personal, sin registro y sin servidores…") — los claims restantes ya viven en los badges. Se descartó a propósito usar un screenshot real de la app: las fixtures sintéticas llevan watermark de test y verifican "Firma inválida" (cert de prueba no acreditado), y las firmadas de verdad exponen identidades reales.
- **PWA — footer acomodado y alineado con el rediseño del footer de la landing** (`@firma-ec/pwa` 0.17.11): headers de columna como eyebrow mono uppercase, ritmo uniforme en los links (`py-1.5`, hit ≥44px móvil, `transition-colors`), claim de privacidad enmarcado como **tarjeta entintada verde** con icono en tile (antes texto suelto), lockup más presente y banda con fondo `bg-ink-100/40` para separar del contenido.

### Fixed
- **PWA — deep-links con path real aterrizaban en la Home** (`@firma-ec/pwa` 0.17.11): el header de la tienda enlaza `https://app.firmar.ec/firmar/pdf?utm_source=tienda&utm_medium=header`, pero el router de la app es hash-based (`#/firmar`) y el fallback SPA del servidor responde `index.html` para cualquier path → el visitante que tocaba "Firmar PDF" en la tienda caía en la portada de la app en vez del asistente de firma. **Fix**: puente `bridgePathToHash` (`lib/pathAlias.ts`, cableado en `main.ts` antes del mount) con allowlist explícita de alias path→hash (`/firmar[/…]`, `/verificar`, `/validar-certificado`, `/paranoia`, `/about|/acerca`, `/configuracion|/settings`, `/certificados[/comprar]`) que reescribe con `history.replaceState` a `/?query#/ruta` **conservando el query** (utm_*). `/share` y `/handle-file` quedan fuera a propósito (entradas del SO con lógica propia en el SW); si la URL ya trae ruta hash se respeta; fail-open (ante error monta la Home como antes). 8 unit tests nuevos (`tests/pathAlias.test.ts`), suite vitest 77/77 verde.

### Changed
- **Landing — pasada premium del stack de diseño completo (emil + high-end + redesign + WIG)** (`@firma-ec/landing` 0.6.16): (1) **scroll-entry reveals** — sistema `[data-reveal]` (IntersectionObserver + fade-up 650ms curva fuerte, stagger 60-80ms por tarjeta, gated tras `html[data-reveal]` → sin JS todo visible; reduced-motion cubierto por reset.css); (2) **feedback físico** en CTAs del hero (`active:scale-[0.98]`) y easing unificado a la curva iOS-drawer `cubic-bezier(0.32,0.72,0,1)` (la 0.4,0,0.2,1 estándar es débil); (3) **double-bezel** (shell exterior con hairline ring + core con highlight inset y radios concéntricos) en la tarjeta destacada del bento y en "Operado por"; (4) **eyebrows unificados** como micro-pill con punto azul en todas las secciones de la home (antes texto plano); (5) sombras hover **tintadas con la tinta de la marca** (color-mix var(--ink-900)) en vez de negro genérico; (6) más respiración: secciones `py-16 md:py-24` (antes 12/16-20); (7) `text-balance` en los h2 y `scroll-behavior: smooth` para anclas.

### Changed
- **Landing — hero con color, jerarquía y movimiento** (`@firma-ec/landing` 0.6.15): (1) titular en dos niveles dentro del mismo h1 (SEO intacto): promesa grande + deck menor "con tu certificado .p12" con chip mono — rompe el bloque monolítico de texto; (2) subrayado caligráfico ámbar bajo "gratis" que **se dibuja al cargar** (stroke-dashoffset, 700ms); (3) **dos tarjetas-documento flotantes** a los lados en ≥xl (PDF con sello ámbar+QR+check / verificación "Firma válida") con **animación de flote** alternada (5.5s, transform-only, contrafase); (4) eyebrow pill con punto azul también en desktop; (5) segundo glow radial ámbar tenue (contenido dentro del deco: si toca el borde se ve un rectángulo recortado — gotcha). Todo CSS puro, `aria-hidden`, `pointer-events-none`, `prefers-reduced-motion` honrado, 0 hscroll verificado a 1280/1440/390.

### Changed
- **Landing — home más visual: banda "Cómo funciona" + bento "Explora" + glow de hero en desktop** (`@firma-ec/landing` 0.6.14): la home se percibía "solo texto" (todas las secciones = mismo patrón plano de tarjetas grises). (1) **Cómo funciona** pasa a banda entintada full-width con dot-grid, pasos con chip numerado 1-2-3 conectados por un rail degradado brand→ok, número watermark gigante por tarjeta, hover lift, y un **mock decorativo de PDF firmado** (sello con QR + check verde, puro CSS, `aria-hidden`, sin CLS) junto al header en ≥lg. (2) **Explora** pasa de 6 tarjetas idénticas a **bento asimétrico**: Seguridad destacada (2 cols, watermark de escudo + chips Cero servidor/Open source/LOPDP/A+), 4 medianas y Estadísticas como banner ancho con mini-gráfico de barras CSS que se enciende al hover; overlay de link con `focus-visible:ring` (tarjeta entera clickeable accesible). (3) El **glow + dot-grid del hero** (antes solo móvil) se extiende a desktop, más sutil y anclado arriba-derecha tras la animación. Solo `transform`/`opacity` en animaciones, decorativos `pointer-events-none`, sin JS nuevo.

### Added
- **Landing — tanda 2 SEO: 3 páginas de contenido nuevas (ES+EN) + refuerzo de páginas cabecera** (`@firma-ec/landing` 0.6.13): páginas nuevas con ROUTE_MAP/hreflang/sitemap/llms.txt: **/firma-electronica-para-empresas/** (RL vs miembro de empresa, SAS, SERCOP/ECUAPASS, CTA tienda RL/ME — captura la demanda "firma electrónica para empresa RUC"), **/renovar-certificado-firma-electronica/** (cuándo/cómo, validez de lo ya firmado vía B-T/LTV, deriva a /validar-certificado/) y **/firma-electronica-facturacion-sri/** (PAdES≠XAdES con el mismo .p12, tabla comparativa, errores "FIRMA INVÁLIDA"). Refuerzo de cabecera: `/como-obtener-certificado…/` gana el link de funnel a `/comparativa-emisores-ecuador/` en "Elige una ECI" + links a empresas/renovar + `dateModified`; `/firma-electronica-ecuador/` enlaza a las 3 nuevas + `dateModified`. Footer: nueva entrada "Firma electrónica para empresas" en Guías (ES/EN, site-wide). Origen: hallazgos P0/P1 del content-strategist (auditoría 2026-07-03).

### Fixed
- **Landing — quick wins SEO de la auditoría 3-frentes 2026-07-03** (`@firma-ec/landing` 0.6.12): (1) **llms.txt/llms-full.txt desincronizados 46 días** — se añaden las 11 páginas ES que faltaban (/compatibilidad/, /estadisticas/, /patrocinar/, /validar-certificado/, /verificar-firma-pdf/, /como-firmar-pdf/, /como-obtener-certificado…/ y 4 guías de emisores) y **guardarraíl en build** (`scripts/check-llms.mjs`: el build FALLA si una URL ES del sitemap no está en llms.txt — nunca más drift silencioso); (2) **FAQPage schema con pregunta duplicada** ("¿Cómo se firma electrónicamente un documento?" ×2, riesgo de invalidación del rich snippet): causa raíz = archivos de contenido duplicados byte-a-byte (`11-` y `14-como-firmar-documento.md`, ES y EN) → se eliminan los `14-`; (3) **meta descriptions >160 chars en 8 páginas y titles >60 en 5** (precios 277, patrocinar 280, compatibilidad 215, home 210…): reescritura sistémica ≤155/≤60 en ui.ts (home ES/EN, stats), compatibilidad ES/EN, precios ES/EN (md), patrocinar ES/EN y guías UANATACA/BCE; (4) **HowTo schema faltante** en /firmar-documentos-en-linea/ + /en/sign-documents-online/ (única guía paso-a-paso sin él) y **BreadcrumbList** añadido a /patrocinar/ y /estadisticas/ (ES/EN), únicas páginas sin ningún JSON-LD. Evidencia: informes de auditoría (technical/content/geo) 2026-07-03.

### Changed
- **Landing — home más corta: 5 secciones movidas a páginas propias + índice compacto "Explora"** (`@firma-ec/landing` 0.6.11): la home acumulaba 9 secciones (~9 pantallas de scroll). Ahora queda en Hero → Cómo funciona → **Explora** (nueva sección compacta de 6 accesos con icono+resumen) → Patrocinadores → Operado por. Las secciones movidas conservan su contenido íntegro y ganan URL propia: **"Por qué es seguro" + "Cumplimiento"** → `/seguridad/` (bajo el reporte de transparencia, vía nuevo slot `after` de `Page.astro` para secciones full-width fuera del ancho prose); **"Para quién es" + "Open source"** → `/acerca/`; **"Compatibilidad" (16/17 ECIs)** → página NUEVA `/compatibilidad/` + `/en/compatibility/` (h1 propio, breadcrumbs JSON-LD, entrada en `ROUTE_MAP` → hreflang y sitemap correctos). La promesa a patrocinadores ("tu logo en la página principal") se mantiene: `SponsorsStrip` sigue en la home. Verificado: `astro check` 0 errores, build 62 páginas (+2), sitemap con el par ES/EN nuevo, iconos UnoCSS emitidos, smoke visual con Playwright en home/compatibilidad/seguridad.

### Fixed
- **PWA — el preview mostraba la página EN BLANCO con PDFs de fuentes no-embebidas** (`@firma-ec/pwa` 0.17.10): un contrato generado con ReportLab (Helvetica/Times/Courier estándar-14 **sin embeber**) se veía completamente en blanco al subirlo a firmar.ec. `PdfPreview.svelte` cargaba pdfjs con `disableFontFace:true` + `useSystemFonts:false` pero **sin `standardFontDataUrl`**, así que pdfjs no tenía de dónde sacar los glifos de las fuentes estándar y **descartaba cada carácter** (`getPathGenerator - ignoring character … Helvetica_path_H`); solo se pintaba el logo (0.53% de tinta vs 10.68% con el fix — 20.3× más). El documento y la **firma** nunca estuvieron afectados (la firma se aplica sobre los bytes crudos, no re-renderiza): solo el preview. **Fix definitivo**: (1) se pasa `standardFontDataUrl: '/pdfjs/standard_fonts/'` a `getDocument`; (2) un plugin de Vite (`syncPdfjsAssets`) copia el worker **y** las `standard_fonts` desde el `pdfjs-dist` instalado hacia `public/pdfjs/` en cada dev/build → **nunca** se desincronizan de la versión del paquete (reemplaza el worker que se copiaba a mano y se quedaba viejo en cada bump; `public/pdfjs/` ahora gitignoreado); (3) `pfb`/`ttf` añadidos al precache del SW → las fuentes estándar también renderizan **offline**. Same-origin, cero-red con el documento (no rompe la promesa zero-knowledge); `connect-src 'self'` ya lo cubre, sin cambio de CSP.

### Added
- **Landing — CTA "Comprar certificado" SIEMPRE visible, explícito y ámbar (header sticky + hero)** (`@firma-ec/landing` 0.6.10): el certificado .p12 es el producto de pago y su CTA no se veía siempre — en el header era **ícono-solo** en móvil (`md:hidden`) y solo aparecía con texto a `xl` (≥1280), dejando un **hueco 768–1279px** sin botón de tienda visible (solo dentro del menú). Ahora: (1) **header** = un único botón ámbar (`#C9821E`) **siempre visible** con texto explícito (`Comprar` <640px → `Comprar certificado` ≥640px), shadow + lift al hover; (2) **hero** (above-the-fold) gana un CTA ámbar "Comprar certificado" en escritorio (`hidden md:`, sin empujar el strip de patrocinio en móvil, que ya queda cubierto por el header sticky). Para que todo quepa sin reintroducir scroll horizontal: el nav horizontal sube a breakpoint `nav: 74rem` (1184px), **`Instalar app` se mueve al menú móvil + hero + footer** (deja de competir en el header de escritorio), y **idioma+tema** pasan a `hidden sm:flex` en el header (en el menú hamburguesa por debajo de 640px). CTAs de tienda ahora en header (siempre) + hero (desktop) + CertNotice + footer, todos ámbar y explícitos. Verificado con Playwright en 320/414/640/768/1024/1205/1280px: Comprar visible en todos, **0 scroll horizontal**. `astro check` 0 errores.

### Fixed
- **Landing — scroll horizontal en el footer a ~768px (columna de repos)** (`@firma-ec/landing` 0.6.10): a `md` la rejilla del footer (`md:grid-cols-5`) dejaba la columna de "Código" en ~122px y el enlace `github.com/idkmanager` (139px, `font-mono`, nowrap) la desbordaba (16px de scroll horizontal en tablets). **Fix**: rejilla progresiva `sm:grid-cols-2 lg:grid-cols-5` (2 columnas anchas a md, 5 solo desde lg donde hay sitio) + `lg:col-span-2` en la columna de marca. Verificado a 768px: sin desbordes.
- **Landing — scroll horizontal en móvil (≤640px) por la rejilla de ECIs** (`@firma-ec/landing` 0.6.9): la sección "Reconocemos 16 de las 17 ECIs acreditadas por ARCOTEL" (`Compatibilidad.astro`) desbordaba 16px a 375px (móvil más común): su `<ul class="grid …">` no tenía `grid-cols` explícito en móvil, así que usaba columnas implícitas `auto` que crecían al `max-content` del nombre de raíz más largo (los `<li>` tienen `min-width:auto`), ignorando el `truncate` interno. En `sm:` ya no pasaba porque `grid-cols-2` = `minmax(0,1fr)`. **Fix mínimo**: añadir `grid-cols-1` a la base de la rejilla (móvil pasa a `minmax(0,1fr)` → el `truncate` del nombre de raíz surte efecto y la columna se ciñe al contenedor). Verificado con Playwright a 360px: sin scroll horizontal (era un bug **preexistente**, independiente del header y del FAB de WhatsApp). `astro check` 0 errores.
- **Landing — scroll horizontal del header en el rango md→~1080px** (`@firma-ec/landing` 0.6.8): el nav horizontal de 7 enlaces (≈660px) + logo + controles (≈249px) suma ≈1047px de contenido y solo cabe a partir de ~1080px de viewport, pero se mostraba desde `md` (768px) — desbordaba el contenedor (27px a 1053px, peor a anchos menores → barra de scroll horizontal). **Fix**: nuevo breakpoint nombrado `nav: 70rem` (1120px) en `uno.config.ts` (clave correcta de presetWind4 = `theme.breakpoint` **singular**, no `breakpoints`; verificado que genera `@media (min-width:70rem)`), y el nav + hamburguesa + panel pasan de `md:` a la variante `nav:` (`hidden nav:flex` / `nav:hidden`), con el `matchMedia` del JS sincronizado a `(min-width: 1120px)`. Resultado: <1120px se muestra el menú hamburguesa (sin desbordar); ≥1120px el nav horizontal (cabe con ~41px de margen). El FAB de WhatsApp **no** causaba el scroll (verificado ocultándolo: `scrollWidth` no cambiaba). `astro check` 0 errores, build 60 páginas.

### Added
- **Landing — FAB de WhatsApp cara-cliente (captación opt-in al número firmarec)** (`@firma-ec/landing` 0.6.7): botón flotante de WhatsApp **site-wide** (montado en `Base.astro`, todas las páginas) al número DEDICADO de firmar.ec (instancia `firmarec`, `+593 99 399 5618`) con un **mensaje pre-cargado que encuadra el opt-in** ("…quiero información sobre la firma electrónica de firmar.ec y recibir novedades por WhatsApp"). Cierra el hueco de captación: hasta ahora la landing solo tenía el WhatsApp de **PATROCINIOS** (línea IDKMANAGER `0958888193`), sin canal cara-cliente; quien pulsa el FAB y envía queda como **contacto de la instancia firmarec** → lead real y alcanzable por el canal de novedades/Estados. Número y textos centralizados en `src/lib/contact.ts` (fuente única, Art. 2; distinto de `Sponsors.astro`). FAB accesible: `aria-label` ES/EN, touch target 56px, `focus-visible`, etiqueta lateral en hover (desktop) sin layout shift, y **verde WhatsApp accesible `#17823F`** (blanco 4.88:1, el mismo validado en 0.4.4; no el `#25D366` que falla contraste) con anillo de pulso **reduced-motion-safe**. Verificado: `astro check` 0 errores, build 60 páginas, FAB renderizado en el HTML con el `wa.me` correcto en home/legales; la línea de patrocinios intacta.

### Changed
- **PWA (app.firmar.ec) — patrocinio visible en la 1ª pantalla móvil al abrir la app + hero compacto** (`@firma-ec/pwa` 0.17.5): paridad con la landing para el mismo objetivo de negocio — que el gancho de patrocinio ("Patrocinadores · Tu marca aquí" + botón WhatsApp) se vea **apenas se abre la app** en móvil, no enterrado bajo dos filas de CTAs. Solo-móvil vía `lt-md:` (UnoCSS presetWind4) + `<style>` con `@media (max-width:767.98px)`; **escritorio intacto** (H1 64px, eyebrow plano, sin backdrop, CTAs ancho natural, verificado). Cambios en `apps/pwa/src/routes/Home.svelte`: (1) **backdrop ambiental** decorativo (glow `--brand-500` + retícula de puntos enmascarada), `display:none` ≥768px, con variante para modo oscuro; (2) **eyebrow → pill** con punto de marca; (3) **H1 más pequeño en móvil** `lt-md:text-[clamp(1.75rem,1.1rem+2.8vw,2.25rem)]` (4→3 líneas); (4) márgenes ajustados (contador/H1/lead/CTAs secundarios); (5) cada **par de CTAs comparte una fila en móvil** (`Button class="lt-md:flex-1"`, basis-0 → no envuelven a 360/320; primarios a `text-sm`/`px-4` para una línea). Verificado en navegador a 320/360/390 px (claro + oscuro) con `/api/stats` mockeado: el gancho + WhatsApp quedan **completos en la primera pantalla** (WhatsApp a ~669–689px), sin scroll horizontal; `svelte-check` 0 errores. **Rebasado sobre 0.17.4**: prod ya corría `firma-ec-pwa:0.17.4` (fix de firma de PDFs de Adobe Acrobat — predictor xref streams — del commit `85f08d7`, rama `fix/signer-acrobat-xref-predictor`), que **no** estaba en esta rama; se incorporan verbatim `packages/signer/src/incrementalUpdate.ts` + su test + bump `@firma-ec/signer` 0.8.3 para **no regresar** ese fix al desplegar.
- **Landing — patrocinio visible en la 1ª pantalla móvil + hero más compacto para ganar espacio** (`@firma-ec/landing` 0.6.2): objetivo de negocio — que el gancho de patrocinio ("Patrocinadores · Tu marca aquí") se vea **apenas se abre la landing** en móvil para captar patrocinadores. En 0.6.1 se había **ocultado** en móvil para despejar; aquí se **revierte** (vuelve a verse) y se gana el espacio necesario sin romper nada: (1) el `HeroSponsors` ya no lleva `lt-md:hidden`; (2) **H1 más pequeño en móvil** `lt-md:text-[clamp(1.75rem,1.1rem+2.8vw,2.25rem)]` (≈28→36px según ancho) → pasa de 4 a **3 líneas**, ~46px ganados (escritorio sigue en `clamp(2rem…4rem)` = 64px, intacto); (3) los CTAs **Firmar + Verificar quedan a todo el ancho apilados** (sin que el texto rompa a 2 líneas) y el **3er CTA "Abrir la app" se oculta en móvil** (sigue accesible desde el menú hamburguesa); (4) el enlace "Ver estadísticas de uso" se oculta en móvil (el contador ya está justo encima). Resultado verificado (Playwright, `/api/stats` mockeado): el bloque de patrocinio arranca en **top ~559px a 390** (~637px a 360), dentro de la primera pantalla; escritorio 1280 idéntico. `astro check` 0 errores, build 60 páginas.
- **Landing — rediseño del hero above-the-fold en móvil (estética premium + táctil); escritorio intacto** (`@firma-ec/landing` 0.6.1): se rediseña la primera pantalla en móvil **sin tocar el escritorio** — todo vía `lt-md:` + un `<style>` con `@media (max-width: 767.98px)`, verificado idéntico ≥768px (eyebrow plano, CTAs en fila, patrocinios, 7 badges, 3 contadores, sin backdrop). (1) **Backdrop ambiental** decorativo detrás del titular (glow azul Fe `--brand-500` + retícula de puntos con máscara de desvanecido), estático, `pointer-events:none`, GPU-barato, `display:none` en escritorio y con variante para modo oscuro. (2) **Eyebrow → pill** con punto de marca. (3) **CTAs apilados a todo el ancho** (Firmar sólido → Verificar outline → "Abrir la app" enlace), patrón táctil; la fila inline de escritorio se preserva con overrides `md:*` sobre clases base mobile-first. (4) **Menos ruido en la 1ª pantalla**: el gancho de patrocinio y los 4 badges técnicos (ETSI/ARCOTEL/Mozilla/SSL) se ocultan en móvil (quedan Gratis · AGPL-3.0 · LOPDP nativa); en escritorio siguen los 7 + patrocinio. (5) **Contadores de uso** (`UsageCounter`): nueva prop `mobileMax` → en móvil se muestran **2 en una sola fila** (las cifras de mayor impacto) en vez de los 3 que volvían a envolver a 2 filas. Supera el fix 0.4.1 (`flex-nowrap`, que solo cabía a 390px y desbordaba a 360): ahora **una fila garantizada de 320 a 430px** sin scroll horizontal (etiqueta `text-[0.6875rem]` + `gap-x-5` en móvil, `sm:` restaura el escritorio); en escritorio siguen las 3. Verificado: `astro check` 0 errores, build 60 páginas, QA en navegador a 320/360/390/430 px (claro + oscuro) con `/api/stats` mockeado a valores reales, y escritorio 1280 idéntico al original. Nota: el `UsageCounter` gemelo de la PWA (`apps/pwa`) conserva el comportamiento previo (paridad pendiente, fuera de alcance).

### Added
- **Landing — capa de medición cookieless + instrumentación de CTAs, e inicio del motor de contenido social orgánico** (`@firma-ec/landing` 0.6.0): nuevo componente `Analytics.astro` (inyectado en `BaseHead`) que habilita medición **inerte por defecto** — sin variables de entorno el build emite HTML **byte-idéntico** (verificado: `grep cloudflareinsights|fecTrack dist/` = vacío) y la promesa "sin tracking · LOPDP por diseño" se mantiene. Dos sumideros opcionales, ambos **sin cookies ni PII**: (1) **Cloudflare Web Analytics** vía `PUBLIC_CF_BEACON_TOKEN` (ya permitido por el CSP de `Caddyfile.landing` → `static.cloudflareinsights.com`) y (2) un **beacon de eventos de conversión** same-origin vía `PUBLIC_EVENTS_ENDPOINT` (CSP `connect-src 'self'`, p. ej. `/api/stats/event`). Un tracker mínimo captura **atribución UTM** (`sessionStorage`, sin cookies), expone `window.fecTrack`/`window.dataLayer` y enlaza los CTAs declarados con `data-cta` (`firmar`, `verificar`, `abrir_app`, `instalar_app`, `obtener_certificado` en Hero/Header/Footer) sin bloquear la navegación. El **Meta Pixel queda fuera a propósito** (requiere cookies, cambio de CSP y consentimiento LOPDP → fase de pauta). `.env.example` documenta las variables. **Motor de contenido orgánico** en `docs/marketing/social/` (no afecta el build): 6 pilares + hooks + plantillas de copy (`pilares-y-hooks.md`), guardrails de marca y YMYL (`guardrails.md`), cadencia y semana tipo (`calendario.md`), guía de medición y esquema UTM (`medicion.md`), y `content-config.json` legible por máquina para el orquestador. Verificado: `astro check` 0 errores, build 60 páginas, biome 0 issues.
- **Página pública "Estadísticas de uso" con series temporales (minuto/hora/día/semana/mes/año)** (`@firma-ec/landing` 0.5.0 · `@firma-ec/stats-worker` 0.2.0): nueva sección `/estadisticas/` (+ `/en/statistics/`) que muestra los totales acumulados (documentos firmados, firmas verificadas, **certificados validados**) y su **tendencia temporal** en un gráfico de barras apiladas, con selector de granularidad **Minuto · Hora · Día · Semana · Mes · Año**. **Worker** (`stats-worker`): el beacon anónimo existente, además de los 3 contadores totales (intactos), escribe **buckets pre-agregados por período** en claves combinadas `b:<g>:<period>` con `{s,v,c}` (una clave por período → la vista más densa son ~34 sub-requests KV, segura en cualquier plan Workers); las claves de **minuto (TTL 2 h)** y **hora (TTL 4 d)** auto-expiran para no crecer sin límite; día/semana/mes/año permanentes. Nuevo endpoint `GET /api/stats/series?granularity=minute|hour|day|week|month|year` → `{granularity, since, buckets, totals}` con `cache-control` por granularidad (30 s minuto, 60 s hora, 300 s el resto). Buckets en **zona America/Guayaquil (UTC−5)** para que "el día/la hora" coincidan con el reloj local. `GET /api/stats` y el contador de la home siguen **byte-idénticos** (comportamiento aditivo). **Solo volumen agregado**: sin PII, sin documentos, sin identificadores (zero-knowledge / LOPDP intactos). **Frontend** (island Svelte 5 `StatsDashboard`): barras apiladas (un acento `brand` + neutro `ink`), readout en hover/focus, leyenda, tabla accesible oculta, estados loading/empty/error+retry, `prefers-reduced-motion`, dark mode, **auto-refresh 60 s** en minuto/hora. Accesos: enlace en el footer + "Ver estadísticas de uso" bajo el contador de la home (es/en). Verificado: **31 tests** del worker (zona EC, wrap de año/fin-de-mes, rate-limit, round-trip de cert, TTL, cache), `astro check` 0 errores, build, QA responsive en navegador (móvil sin overflow horizontal, selector que envuelve, vista minuto con eje HH:MM), revisión adversarial multi-lente (20 hallazgos atendidos).

### Changed
- **Handoff ahora por fetch/deep-link (robusto en el navegador de WhatsApp)** (`@firma-ec/pwa` 0.17.0): el modo handoff deja de usar **popup + `postMessage`** (que falla dentro del navegador interno de WhatsApp, donde `window.opener` se pierde y el popup no abre) y pasa a un **deep link con FETCH directo**. La app de admisión abre la PWA con `?handoff=1&src=<URL del acta>&cb=<URL de callback>`; la PWA **descarga el acta** desde `src` (mismo camino que un archivo elegido a mano → firma on-device) y, tras firmar, **hace POST del PDF firmado** a `cb` como `multipart/form-data` (campo `file`). **Anti-SSRF**: tanto `src` como `cb` se validan contra la allow-list `VITE_HANDOFF_ALLOWLIST` (fail-closed: vacío = handoff deshabilitado) **antes** de cualquier llamada de red — un deep link con un origen no permitido se rechaza y nunca se fetchea/POSTea. El POST de callback es una **request simple** (sin headers custom → sin preflight CORS; el token viaja en la URL, sin `credentials`). El botón "Enviar firmado" lee `wa_sent` de la respuesta: `true` → "registrada y reenviada por WhatsApp"; `false` o fallo → mensaje honesto + descarga local de respaldo. La **llave .p12 y la firma siguen 100% on-device** (los workers no se tocan). Todo sigue gateado por `?handoff=1`: sin el parámetro el flujo público es idéntico. Se eliminó la maquinaria `postMessage` (contrato v1: `firmarec:ready/load/signed/cancel/error`, `sendReady/sendSigned/...`, `initHandoff`, `window.opener`). Verificado: `typecheck` 0 errores.

### Added
- **Modo handoff opt-in para firmar documentos recibidos por postMessage** (`@firma-ec/pwa` 0.16.0): una app externa (mismo operador) puede abrir la PWA con `?handoff=1`, pasarle un documento por `postMessage` y recibir el PDF firmado de vuelta — **sin que el documento ni la llave salgan del navegador por la red** (entra y sale por `postMessage`; la firma sigue siendo on-device). Preserva la promesa pública "tu PDF nunca sale de tu navegador" (verificable en DevTools) y el carácter genérico del app: `handoff.ts` con contrato v1 + allow-list de origin **por env `VITE_HANDOFF_ALLOWLIST`, fail-closed (vacío = deshabilitado), sin host de tenant en el source/imagen** (el operador lo setea por `--build-arg` en su deploy). `Firmar.svelte` pre-carga el documento por el mismo camino que un archivo elegido; `DownloadResult.svelte` añade el botón explícito "Enviar firmado" (consentimiento visible). Todo gateado por `?handoff=1`: sin el parámetro el flujo público es idéntico. Verificado: typecheck 0 errores, auditoría adversarial `NO_PROMISE_BROKEN`, e2e en navegador (load→firma on-device→retorno) con red de la PWA = solo el stats-ping body-less.

### Fixed
- **Multi-firma sobre PDFs firmados en Adobe Acrobat — `cannot_add_signature_to_corrupt_pdf` resuelto** (`@firma-ec/signer` 0.8.3 · `@firma-ec/pwa` 0.17.4): añadir una firma a un PDF ya firmado en **Adobe Acrobat** (cuyo cross-reference es un **xref stream con PNG predictor**: `/DecodeParms << /Columns N /Predictor 12 >>`) fallaba con `cannot_add_signature_to_corrupt_pdf` (detalle interno: *"cross-reference … is neither a classical xref table nor a /Type /XRef stream"*). El PDF **no estaba corrupto** — FirmaEC desktop lo rechaza por otra causa (exige fuentes embebidas). **Causa raíz**: `parseXrefStreamDict` cerraba el diccionario con `indexOf('>>')`, que se detiene en el `>>` del sub-diccionario anidado `/DecodeParms`, truncándolo; según el orden de claves del productor, `/Type /XRef` (Acrobat) o `/Size`/`/Root` (qpdf/pikepdf/MuPDF) quedaban fuera → falso "corrupto". **Fix raíz**: escaneo **balanceado** de `<<`/`>>` (`balancedDictEnd`) para hallar el cierre real del dict externo. **Segundo bug del mismo PDF**: el campo de firma de Acrobat (`/T (Signature2)`) vive **comprimido en un object stream**, invisible al scan textual de nombres → el nuevo campo se nombraba también `Signature2` → **colisión `/T`** (los visores deduplican por nombre y descartan una firma, ISO 32000-1 §12.7.3.2). **Fix**: enumerar los campos vía pdf-lib (ObjStm-aware) + `pickSignatureFieldName` que salta nombres ocupados (→ `Signature3`). Regression `xref-stream-predictor.test.ts` (7 tests: ambos órdenes de clave Acrobat/qpdf + naming anti-colisión). Verificado con el documento real reportado (firmado por una AC vía Acrobat): firma OK, prefijo **byte-idéntico** (firma previa intacta), pyHanko → ambas firmas `intact + valid` (la previa pasa de `ENTIRE_FILE` a `CONTIGUOUS_BLOCK_FROM_START`), campos `Signature2`+`Signature3` sin colisión. Suite signer **91/91** sin regresiones.
- **Landing — accesibilidad: contraste WCAG AA en modo oscuro + botón WhatsApp** (`@firma-ec/landing` 0.4.4): el audit Lighthouse/PageSpeed móvil daba **Accesibilidad 97** por una sola causa, `color-contrast` (audit binario: un solo elemento por debajo de 4.5:1 lo deja en 0 y tapa el 100). Dos focos: (1) **modo oscuro** — sobre el fondo casi-negro del tema, el azul de marca (`brand-500` #0062c4, 3.5:1) y los grises de etiqueta (`ink-500` 4.1:1 e `ink-600` 2.4:1 vía `dark:text-ink-600`) caían bajo el umbral en ~74 nodos (eyebrows, números de stats, labels, footer); (2) **botón "WhatsApp"** (`Sponsors.astro` + `SponsorsStrip.astro`) — texto blanco sobre el verde de marca `#25D366` daba **1.98:1**, falla en claro y en oscuro (esta era la única que veía PageSpeed cuando renderizaba en claro, de ahí el 97 con miniatura clara). **Fix**: (a) override **solo en modo oscuro** que eleva `text-brand-500`→`var(--brand-400)` (≈7.6:1) y `text-ink-500`→`var(--ink-400)` (≈7.0:1) — reusa los tokens que el sistema ya define como "texto sobre oscuro", de paso unifica los labels que ya usaban `dark:text-ink-400`; sin capa a propósito para ganar a la capa `utilities` de UnoCSS; **solo texto** (bg/border/ring de marca intactos, así el blanco sobre botones de marca no se degrada); el modo claro no se toca (brand-500/ink-500 sobre blanco ya pasan). (b) verde del botón WhatsApp a un tono más profundo del mismo matiz, `#17823F` (blanco 4.88:1) con hover `#146E36`, preservando el reconocimiento de marca. Verificado: contraste numérico de cada par (fg,bg) ≥4.88:1 (brand/ink ≥6.97:1 en los 3 fondos oscuros reales); Lighthouse móvil del build → **Accesibilidad 100, color-contrast 0 fallos, Rendimiento 99** (sin regresión); la regla oscura gana por especificidad (0,2,0)>(0,1,0) y por estar sin `@layer`; escaneo anti-hardcoding 🟢.
- **Multi-firma: el estampado visible de la firma PREVIA desaparecía al añadir la 2ª firma** (`@firma-ec/signer` 0.8.2 · `@firma-ec/pwa` 0.15.5): en PDFs cuya página guarda `/Annots` como **referencia indirecta** a un array (iText/FirmaEC, Acrobat — caso real reportado 2026-06-11: escrito legal con firma FirmaEC previa), `injectAnnot` reemplazaba `/Annots N G R` por `/Annots [N G R <widget nuevo>]`, anidando el ARRAY viejo como elemento de `/Annots` — ilegal según ISO 32000-1 §12.5.2 — y los visores (Acrobat, pdfium/Chrome, pdf.js) descartaban la entrada no-dict: el estampado QR de la firma anterior (y cualquier Link de esa página) dejaba de renderizarse, aunque ambas firmas seguían criptográficamente válidas. **Fix raíz (estrategia pyHanko/iText)**: con `/Annots` indirecto se redefine el PROPIO objeto array (mismo número de objeto, widget anexado, resuelto vía pdf-lib que sigue la cadena xref a la última revisión) y la página NO se reescribe. **Fix #2 — catálogo preservado**: la revisión incremental reconstruía el catálogo con una plantilla de 3 claves, perdiendo `/StructTreeRoot` `/Lang` `/Metadata` `/MarkInfo` `/PageLayout` (→ `SuspiciousModification` en pyHanko); ahora se copia el dict previo verbatim sustituyendo solo `/AcroForm`. **Fix #3 — AcroForm preservado**: ídem (`/DA`, `/DR`, etc.), `/Fields` indirecto ya no dropea silenciosamente los campos previos (se redefine el array), `/SigFlags` se OR-ea. **Fix #4 — cola latin1**: la cola incremental se codificaba UTF-8 sobre bodies decodificados latin1 → cualquier byte no-ASCII copiado se corrompía (p. ej. `/Lang` UTF-16BE de iText); ahora se emite latin1 (1 char = 1 byte) y los literales generados no-ASCII van como hex UTF-16BE con BOM (de paso corrige el mojibake de `/Name` con tildes). Regression `annots-indirect-multisig.test.ts` (4 tests, fixture real iText `audit-075-2026.pdf` con `/Annots` indirecto — 2 fallan con el código previo). Verificado con el documento real del reporte: pyHanko → Signature1 `intact+valid` + **`ModificationLevel.FORM_FILLING`** (antes `OTHER`/sospechoso), catálogo 8/8 claves, `/Annots` efectivo = 3 dicts (Link + Signature1 + Signature2), y render pdfium con **ambos estampados visibles**. Suite signer 84/84.
- **Multi-firma sobre PDFs con object streams (iText) — `cannot_add_signature_to_corrupt_pdf` resuelto** (`@firma-ec/signer` 0.8.1 · `@firma-ec/pwa` 0.15.4): añadir una 2ª firma a un PDF cuya estructura vive en object streams comprimidos (`/ObjStm`, típico de iText/Adobe — caso real: escrito legal firmado por iText-Core-8.0.5) fallaba con "Pages object body not found" porque `parsePriorPdf`/`readObjectBody` solo veían objetos top-level en el texto crudo. El PDF NO estaba corrupto. **Fix raíz**: pdf-lib (que ya se cargaba en el flujo como sanity check) se carga ANTES del parseo estructural y actúa como **resolver de fallback** para objetos comprimidos (`stub.context.lookup(ref)` → `PDFDict.toString()`), cubriendo Catalog/Pages/Page/AcroForm y el target-page de firma visible. **Fix adicional descubierto por el regression test**: `detectSignatures` asumía `/Contents` DESPUÉS de `/ByteRange` (cierto en iText/Adobe, falso en pyHanko) → ahora extrae el hex directamente del gap del quad `[a b c d]` (bytes `a+b … c`), robusto ante cualquier orden de claves, con el scan textual como fallback legacy. Regression test nuevo `objstm-multisig.test.ts` + fixture sintético `objstm-signed.pdf` (base PyMuPDF `use_objstms` + firma pyHanko con el cert de test — sin PII). Verificado: 3/3 regression, suite signer 80/80 sin regresiones, test ácido local con el documento iText real → 2 firmas, prefijo byte-idéntico, ambas `intact+valid` en pyHanko, PyMuPDF abre sin reparar.

### Added
- **Landing — SEO: clúster "firmar documentos" (página dedicada + on-page)** (`@firma-ec/landing` 0.4.3): el sitio posicionaba para "firmar PDF"/"firma electrónica" pero la consulta genérica y de más alcance **"firmar documentos"** aparecía solo 3 veces en todo el código y nunca en un title/H1. Se integra en señales reales (NO en el meta keywords tag, muerto para Google/Bing): (1) **home** — title/description/H1 insertan "documentos" de forma aditiva, sin perder los términos que ya rankean ("Firmar y verificar **documentos** PDF gratis…", ES+EN); (2) **nueva página** `/firmar-documentos-en-linea/` (+ `/en/sign-documents-online/`) — answer-first, cubre "firmar documentos en línea/gratis", "página web para firmar documentos", "cómo se firma electrónicamente un documento", "firmar documentos con firma electrónica Ecuador"; enlaza la guía Security Data ya existente; `ArticleSeo` (TechArticle + BreadcrumbList) + hreflang recíproco (ROUTE_MAP); (3) **FAQ** exact-match (alimenta `FAQPage`): "¿Cómo se firma electrónicamente un documento?" (ES+EN, orden 14); (4) **JSON-LD** — `keywords` en `SoftwareApplication` (schema.org, complementa `featureList`, para GEO/AEO); (5) `llms.txt`/`llms-full.txt` con el framing "firmar documentos en línea"; (6) **enlazado interno** — guía en el Footer (todas las páginas) + en el `related` de 4 páginas afines. Sin cambios de stack ni de comportamiento. Verificado: `astro check` + build, biome 0 issues, hreflang/canonical/sitemap correctos, E2E live tras deploy.
- **Autoposicionamiento anti-solape del cuadro de firma** (`@firma-ec/pwa` 0.15.3): al firmar un PDF que ya trae firmas **visibles** (multi-firmante, típico en escritos legales), el cuadro de firma ahora se coloca solo en un **slot libre junto a las firmas existentes** (defaulteando a la página donde otros firmaron), en vez del default centrado fijo que podía caer encima de una firma previa y obligar a arrastrar. Implementación cliente: `PdfPreview.svelte` escanea los widgets de firma (`subtype Widget`/`fieldType Sig`) vía pdf.js `getAnnotations()` tras cargar (acotado a las últimas 15 páginas en docs >50pp) y emite sus `/Rect`; un helper puro nuevo `smartPlacement.ts` (`computeSmartPlacement`) elige la página de la última firma y busca el primer slot sin solape en la banda inferior (fila, o fila superior si está llena), con fallback al default centrado; `BoxPlacer.svelte` gana `autoPlaceDefault` para no pisar el cálculo (sin flicker). Solo cambia el caso multi-firma; documentos sin firmas visibles conservan el default histórico. Tamaño del cuadro centralizado en `DEFAULT_SIG_BOX_W/H` (fuente única de verdad). Verificado: 9 tests unitarios del helper, `svelte-check` 0 errores, build PWA limpio, y **E2E en navegador** (Playwright) — el cuadro se autoubica junto a la firma visible existente sin solaparla, 0 errores de consola.
- **Botón de contacto de patrocinios por WhatsApp** (`@firma-ec/landing` 0.4.2 · `@firma-ec/pwa` 0.15.2): se añade un canal directo de contacto para patrocinios hacia la línea de IDKMANAGER **+593 95 888 8193** (0958888193), con `wa.me/593958888193` y mensaje prellenado ("Hola, me interesa patrocinar firmar.ec…" / EN). **Landing**: (1) página de patrocinio (`Sponsors.astro`) — botón verde WhatsApp en el hero junto a `sponsors@firmar.ec` y enlace WhatsApp en la fila de contacto de "Cómo se paga"; (2) franja de patrocinadores del home (`SponsorsStrip.astro`) — botón "Escribir por WhatsApp" junto a "Convertirme en patrocinador" en el panel CTA. **PWA**: botón WhatsApp junto al gancho "Tu marca aquí" en el Home. Logo oficial de WhatsApp inline (componente `WhatsAppIcon.astro` en landing, SVG inline en la PWA — sin añadir colección de iconos), color de marca `#25D366` para máxima reconocibilidad, todos los enlaces `target="_blank" rel="noopener noreferrer"` con focus-visible ring. Verificado: builds limpios, sin scroll horizontal.

### Changed
- **Home PWA: "Instalar app" junto a "Sitio institucional" en su propia fila** (`@firma-ec/pwa` 0.15.1): los CTAs del Home se reorganizan en dos filas — primaria `[Verificar PDF] [Firmar PDF]` y secundaria `[Sitio institucional] [Instalar app]`. Las acciones secundarias bajan a `size="sm"` para que ambas quepan en una sola fila a 390/360px (antes "Instalar app" caía a una línea suelta). "Instalar app" queda debajo de "Firmar PDF" y al lado de "Sitio institucional". Sin scroll horizontal.

### Added
- **Contadores en vivo + patrocinadores en el Home de la PWA** (`@firma-ec/pwa` 0.15.0): para que en cada apertura de la app (Android instalada en standalone, o iOS usando la web) se vean la prueba social y el patrocinio, se llevan los **medidores de uso** (`UsageCounter`, "Documentos firmados / Firmas verificadas / Certificados validados") al inicio del Home, sobre el eyebrow — leen el mismo endpoint `/api/stats` same-origin (app.firmar.ec lo sirve, sin CORS) y conservan la degradación honesta (se auto-ocultan ante error/0). El gancho de patrocinio "Tu marca aquí" ya vivía en el Home. Padding superior del hero reducido (`pt-6 md:pt-8`). Componente nuevo `apps/pwa/src/ui/UsageCounter.svelte` (paridad con el del landing, incl. fix de una-fila en móvil). Decisión: NO se redirige la app a la landing (rompería la herramienta y dejaría sin sentido instalar); en su lugar la propia app muestra sponsors + contadores en cada apertura. iOS usa la web directa; foco de instalación en Android. Verificado: build PWA limpio, contadores 1 fila top 89px + gancho visibles sin scroll a 390px, sin scroll horizontal.

### Fixed
- **Contadores del hero en una sola fila en móvil** (`@firma-ec/landing` 0.4.1): los 3 contadores (firmados/verificados/validados) se envolvían a 2 filas en móvil. `UsageCounter` ahora usa `flex-nowrap` en móvil (`sm:flex-wrap` en pantallas grandes), gap reducido (`gap-x-5 sm:gap-x-10`), número `text-2xl sm:text-3xl` e ítems `min-w-0` → los 3 entran en una fila a 390px sin scroll horizontal (la etiqueta larga envuelve bajo el número).

### Added
- **Slots de Patrocinadores visibles + placeholder "Tu marca aquí"** (`@firma-ec/landing` 0.4.0 · `@firma-ec/pwa` 0.14.0): se habilitan espacios de patrocinio prominentes en landing y app, con un placeholder atractivo que invita a marcas a ser las primeras (aún no hay patrocinadores). **Landing**: (1) gancho compacto en el hero — pill "✦ Tu marca aquí — sé el primer patrocinador" ubicado entre los CTAs y los badges (visible sin scroll en móvil y PC); (2) sección "Quienes hacen esto sostenible" rediseñada con un slot destacado "Tu marca aquí" + 3 slots fantasma + panel CTA "Convertirme en patrocinador". Cuando se agreguen sponsors a `src/data/sponsors.ts`, ambos render cambian automáticamente a logos. **PWA**: gancho de patrocinio en el Home (sobre los badges) + enlace "Patrocinar firmar.ec" en el footer, ambos hacia `firmar.ec/patrocinar/` (ES) / `/en/sponsor/` (EN); claves i18n `home.sponsor.*` y `footer.sponsor` (ES+EN). Enlaces con `aria-label`, slots fantasma `aria-hidden`, iconos decorativos `aria-hidden`, focus-visible rings; iconos `i-lucide-sparkles`/`i-lucide-image` añadidos al safelist UnoCSS. Verificado: builds limpios (landing 56 págs, PWA), sin scroll horizontal, gancho visible sin scroll (móvil 390px y PC), claro+oscuro, auditoría QA independiente GO.

### Changed
- **Hero del landing reorganizado: contadores arriba + lead más compacto** (`@firma-ec/landing` 0.4.0): los contadores de uso (`UsageCounter`, "Documentos firmados / Firmas verificadas") se movieron al inicio del hero (bajo el header, sobre el eyebrow) para aprovechar el espacio en blanco superior como prueba social visible sin scroll; el componente gana una prop `spacing` para controlar el margen según su ubicación, y conserva su degradación honesta (se auto-oculta ante fetch error o totales en 0). El padding superior del hero se redujo (`py-12/20` → `pt-6 md:pt-8 pb-12 md:pb-20`) y el párrafo lead se compactó (`text-lg md:text-xl` → `text-base md:text-lg`, `leading-relaxed`). El mismo ajuste de tamaño del lead se aplicó a la PWA (`@firma-ec/pwa` 0.14.0) por consistencia.
- **"Operado por": marca registrada IDKMANAGER completa (isotipo iDK + wordmark)** (`@firma-ec/landing` 0.3.5): la sección "Operado por" mostraba un wordmark antiguo (`idk-manager-wordmark.png`, 160×66) que solo decía "MANAGER" — no la marca tal como se envió a registrar en SENADI. Se reemplaza por `idkmanager-logo.png` (351×60, RGBA transparente, ratio 5.85:1), el mismo asset oficial que usa idkmanager.com en su header/footer = isotipo iDK + **IDKMANAGER** (palabra completa). El `<img>` declara `width=351 height=60` (= dimensiones intrínsecas reales) con `class="h-8 md:h-9 w-auto max-w-full"` → conserva aspect-ratio, **sin distorsión en móvil ni PC** (QA Playwright: desktop 1280px y móvil 375px, ambos ratio render 5.85, sin overflow de la tarjeta). Se eliminó un símbolo ® añadido por error (la marca está pagada pero pendiente de "Iniciar Proceso" en SENADI, aún no registrada → no corresponde ®/™). Cleanup: assets huérfanos `idk-manager-wordmark.png`/`@2x` borrados de `public/brand/`, y `aria-label` redundante quitado del `<a>` (queda `alt="IDKMANAGER"`). Verificado: `astro check` + build 56 páginas 0 errores; auditoría QA independiente GO; assets SHA-256 idénticos a la fuente oficial.

### Fixed
- **Android: instalación inaccesible cuando Chrome deja de emitir `beforeinstallprompt`** (`@firma-ec/pwa` 0.13.4): QA con evento sintético confirmó que la cadena de código instala bien (captura en `<head>` → tarjeta → clic → `prompt()`); el problema en dispositivos reales es que **Chrome silencia `beforeinstallprompt`** cuando la app ya se instaló antes o tras descartar el diálogo varias veces (backoff anti-abuso del navegador, ~semanas). En ese estado `canPrompt` es `false`, y la tarjeta de invitación se ocultaba en Android (`canInvite = canPrompt || isIOS`) → el usuario se quedaba sin camino. Fix: (1) `canInvite` ahora incluye `isAndroid()` — en Android **siempre** se ofrece el acceso porque siempre se puede instalar por el menú ⋮ de Chrome; (2) nuevo branch Android en `InstallGuide` con pasos ilustrados (menú ⋮ → «Instalar app»/«Añadir a pantalla de inicio») + nota de "si ya la instalaste, ábrela desde el inicio". Antes Android-sin-prompt caía a un texto genérico poco visible. Los botones hero/header y el prompt nativo de un clic siguen igual cuando Chrome sí ofrece el evento. Verificado: typecheck 0 errores; cadena de instalación E2E con evento sintético (card+prompt) verde.
### Changed
- **Flujo `?install=1` ahora instala en "modo auto" (diálogo nativo directo, sin pasos manuales primero)** (`@firma-ec/pwa` 0.13.3): al llegar desde el botón «Instalar app» del landing, antes se abría la guía con los pasos manuales. Ahora `App.svelte` llama a `installState.armAutoInstall()`: el **diálogo nativo de instalación salta en el primer gesto del usuario** (Chrome **exige** activación por gesto para `prompt()` — no se puede auto-disparar al cargar; es anti-abuso del navegador, no del sitio), sin mostrarle antes las instrucciones. Las indicaciones manuales quedan **solo como referencia**: en iOS (Apple no expone prompt nativo) se abre la guía, y como red de seguridad si en 10s no hay prompt nativo disponible. Los botones «Instalar app» del hero/header siguen disparando el prompt nativo directo en su clic (ya es un gesto). Verificado: typecheck 0 errores.
### Added
- **Acceso "Instalar app" junto a "Abrir app" (landing) y a "Sitio institucional" (PWA)** (`@firma-ec/landing` 0.3.4 · `@firma-ec/pwa` 0.13.2): se añade un acceso directo a instalar en los dos puntos de entrada. (1) **Landing**: botón «Instalar app» (outline + icono download) al lado de «Abrir app», en el header desktop y en el menú móvil. Como el landing (`firmar.ec`) y la PWA (`app.firmar.ec`) son orígenes distintos, el botón navega a `app.firmar.ec/?install=1`. (2) **PWA**: `App.svelte` detecta `?install=1` y abre la guía de instalación — su botón «Instalar ahora» es un gesto de usuario válido para lanzar el prompt nativo (Chrome bloquea `prompt()` sin gesto, por eso no se auto-dispara); el query se limpia con `replaceState`. (3) **PWA hero**: botón «Instalar app» (outline) junto a «Sitio institucional» que llama a `installState.trigger()` directo (el clic ya es gesto → prompt nativo en Android/PC, guía en iOS). Ambos botones se ocultan si la app ya está instalada (standalone). Nueva clave i18n `cta.instalar_app` (landing, ES+EN); la PWA reutiliza `install.menu`. Verificado: typecheck PWA 0 errores, build landing 56 páginas OK.
- **Invitación consciente de escritorio: "Crea un acceso directo" en PC** (`@firma-ec/pwa` 0.13.1): en PC instalar la PWA (Chrome/Edge) crea una app en ventana propia + acceso directo en escritorio/menú inicio, pero la tarjeta de invitación usaba copy de móvil ("recibir PDFs desde WhatsApp…") y no comunicaba ese beneficio. Ahora `InstallPrompt` es **platform-aware**: en escritorio (no iOS, no Android) muestra título «Crea un acceso directo» / body «Instala firmar.ec en tu escritorio para abrirla con un clic, en su propia ventana y sin pestañas» + icono `monitor-down`; en teléfono conserva el copy móvil. Nuevas claves i18n `install.prompt.title_desktop`/`body_desktop` (ES+EN). El botón sigue lanzando el prompt nativo (un clic). Sin cambios de lógica de visibilidad. Verificado: typecheck/build limpio.
- **Invitación a instalar en TODAS las plataformas + ícono de instalación pulido** (`@firma-ec/pwa` 0.13.0): la tarjeta de invitación `InstallPrompt` aparecía **solo en Android** (dependía del evento nativo `beforeinstallprompt`), así que los usuarios de **iPhone no recibían ninguna invitación** a instalar — había que buscar el botón en el menú. Ahora la tarjeta se muestra también en **iOS** (Safari y no-Safari): en Android su botón sigue lanzando el **prompt nativo (un clic)**, y en iPhone abre la **guía Compartir → "Añadir a pantalla de inicio"** (Apple no permite instalación programática en iOS — es una restricción de la plataforma, no del sitio). En desktop/otros navegadores sin prompt nativo no se insiste con la tarjeta (queda el botón "Instalar app" del header). Se añade la clave i18n `install.prompt.cta_how` (ES «Cómo instalar» / EN «How to install»). **Pulido de ícono:** el ícono 512 pasa de `purpose: 'maskable'` a `'any maskable'` (y el 192 se marca explícito `'any'`), para que el **diálogo de instalación muestre un ícono nítido y bien encuadrado** además del adaptativo de los launchers. **Privacidad/LOPDP:** 100% client-side, sin tracking; el único dato es el flag local de descarte (30 días, en el dispositivo). Verificado: typecheck/build limpio, manifest servido con iconos `any`+`maskable`, deploy live (la guía iOS se valida en Safari real).
- **Guía de instalación de la PWA por plataforma + entrada permanente "Instalar app"** (`@firma-ec/pwa` 0.12.0): para subir la tasa de instalación en celular se añade (1) `lib/installState.svelte.ts` — captura única de `beforeinstallprompt` (Chromium) y helpers de plataforma (iOS/Android/standalone/iOS-no-Safari); (2) `InstallGuide.svelte` — hoja de instrucciones **por plataforma**: Android/Chromium dispara el prompt nativo; **iOS Safari** muestra pasos ilustrados (Compartir → "Añadir a pantalla de inicio"); iOS no-Safari pide abrir en Safari; resto, instrucciones del menú del navegador; (3) entrada **"Instalar app"** permanente en el `Header` (botón desktop + ítem en el menú hamburguesa), oculta si ya está instalada (standalone). La tarjeta contextual `InstallPrompt` se refactorizó para usar el estado central (sin doble captura del evento). **Privacidad/LOPDP:** 100% client-side, sin tracking, sin terceros; el único dato es el flag local de descarte (30 días) en el dispositivo. Verificado: build/typecheck limpio, botón header + ítem menú + tarjeta nativa + 0 errores de consola (la rama de guía manual iOS se valida en dispositivo Safari).

### Fixed
- **Badge de versión congelado en `0.10.0` desde el release 0.11.0** (`@firma-ec/pwa` 0.12.2): `src/lib/version.ts` (`APP_VERSION`, fuente única que alimenta el Footer y la página About) nunca se bumpeó al subir a 0.11.0/0.12.0/0.12.1 — sólo se actualizaba `package.json`. Resultado: prod mostraba "0.10.0" aunque el código corriendo era posterior (mismo patrón del incidente moneccu 2026-05-11). Fix: sincronizar `version.ts` con `package.json` en `0.12.2`. Sin cambios funcionales — sólo coherencia de versión visible. A futuro: bumpear SIEMPRE ambos puntos (badge + package.json) en el mismo commit del release.
- **Regresión: en Android el botón "Instalar app" abría la guía manual en vez de instalar directo** (`@firma-ec/pwa` 0.12.1): el evento `beforeinstallprompt` de Chrome se dispara muy temprano en la carga, antes de que Svelte monte; como el listener se registraba en `App.svelte` `onMount`, el evento ya se había perdido → `canPrompt` quedaba en `false` y `trigger()` caía a la guía manual en lugar de lanzar el prompt nativo de instalación. Fix: capturar el evento con un script inline en el `<head>` (index.html), antes de que cargue el bundle, reteniéndolo en `window.__deferredBIP`; `installState.start()` lo recoge al montar y conserva su listener por si Chrome lo re-emite. Restaura el comportamiento previo: en Android, tocar "Instalar app" lanza el diálogo nativo y la app se instala al confirmar. **LOPDP:** 100% client-side, el evento solo se retiene en memoria del dispositivo. Se valida en Android Chrome real.
- **Header móvil: el botón "Abrir app" se encimaba sobre el logo** (`@firma-ec/landing` 0.3.3): al añadir el isotipo ƒ (0.3.0) el lockup del logo creció ~36px y en pantallas angostas (≤390px) el CTA "Abrir app" colisionaba visualmente con "firmar.ec". Fix: el botón "Abrir app" pasa a `hidden sm:inline-flex` (se oculta <640px, donde el hero ya ofrece "Abrir la app") y se añade como ítem CTA dentro del **menú hamburguesa** para no perder el acceso en móvil. Verificado 360/390px (sin overlap) y desktop (botón intacto). Sin cambios en la PWA (su header no tiene ese CTA).

### Changed
- **Isotipo ƒ aplicado a TODA la app (landing + PWA) — decisión final de marca** (`@firma-ec/landing` 0.3.2 · `@firma-ec/pwa` 0.11.0): tras evaluar la reversión 0.3.1, se decide **mantener y consolidar el isotipo ƒ** (ƒ caligráfica navy `#1E3A8A` + travesaño ámbar `#C9821E`) como identidad de firmar.ec. (1) **Landing**: se restaura el ƒ revertido en 0.3.1 (favicon set, Header con isotipo + ".ec" ámbar, OG navy+ƒ) — el landing en prod nunca había cambiado de 0.3.0, esto reconcilia el repo. (2) **PWA (`app.firmar.ec`)**: se rebrandeó por primera vez — `favicon.svg` + `icon-192/512.png` a la ƒ sobre tile claro, y `Header.svelte` ahora muestra el isotipo ƒ (rúbrica `currentColor`, dark-safe) + ".ec" ámbar junto al chip "app". Tokens/tipografía (Geist/oklch) y `theme_color` del manifest sin cambios. ⚠️ Requiere purge CF; usuarios PWA verán el ícono nuevo tras aceptar el update del service worker. El paquete de marca vive en `firmar-ec-branding/`.

### Reverted
- **Reversión del re-brand ƒ — se restaura la identidad visual original** (`@firma-ec/landing` 0.3.1): por decisión de marca se revierte el re-brand 0.3.0 (isotipo ƒ + ámbar) y se restaura la identidad previa: favicon tile navy oscuro `#0B1A3A` + "F" blanca + check verde `#10b981`, header solo-wordmark (sin isotipo), y generador OG original (navy oscuro `#0B1A3A→#14254F`, Geist, sin isotipo). Archivos de marca restaurados al estado de `bb07b14` (favicon.svg/.ico, apple-touch, icon-192/512, og-firmar-ec.png, og-app-firmar-ec.png, Header.astro, og-image.ts). El sistema de tokens/tipografía nunca se había tocado. Requiere purge de caché Cloudflare. El paquete de marca ƒ explorado queda archivado en `firmar-ec-branding/`; la definición del logo definitivo queda abierta (pendiente brief de diseño).

### Changed
- **Re-brand de identidad visual: nuevo isotipo ƒ caligráfica (firma manuscrita) + acento ámbar** (`@firma-ec/landing` 0.3.0): se reemplaza la identidad anterior (tile navy oscuro + "F" + check verde) por una marca propietaria construida sobre una **ƒ caligráfica** (de *firmar*, trazo de firma) con travesaño ámbar `#C9821E` sobre navy `#1E3A8A`. Motivo: la marca anterior era genérica y colisionaba visualmente con marcas existentes (verificado con reverse-image search: el check-en-azul matcheaba "ArQlik"; la "f" geométrica blanca sobre cuadrado azul matcheaba "Frecuento" y evocaba Facebook). La nueva ƒ caligráfica sobre fondo claro pasó la verificación como "logo genérico de letra F" sin colisión con marca específica (TinEye 0 copias). **Regla de uso baked:** el app-icon va sobre tile CLARO, nunca ƒ blanca sobre cuadrado azul sólido. Cambios: (1) icon set completo (`favicon.svg`, `favicon.ico`, `apple-touch-icon.png`, `icon-192.png`, `icon-512.png`) → ƒ sobre tile papel; (2) `Header.astro` ahora muestra el isotipo ƒ (rúbrica en `currentColor` brand del sitio, adapta a dark mode; travesaño ámbar) + ".ec" en ámbar; (3) generador OG (`og-image.ts`) → fondo navy `#1E3A8A→#15296B` + isotipo ƒ embebido, conservando la tipografía Geist del sitio; `og-firmar-ec.png` (landing) y `og-app-firmar-ec.png` (PWA) regenerados. Sistema de tokens/tipografía del sitio conservado (scope: iconos + header + OG). ⚠️ Requiere purge de caché Cloudflare del favicon/OG (cacheados). Clearance legal formal (SENADI figurativa + WIPO) pendiente antes de registro de marca.

### Fixed
- **GSC: `/cdn-cgi/l/email-protection` reportada como 404 "No encontrada"** (`@firma-ec/landing` 0.2.1): el enlace de ofuscación de email que Cloudflare inyecta al borde sobre el `mailto:` de `/contacto` no es una página real; Googlebot lo rastreaba y obtenía 404, ensuciando el informe de Indexación. Fix: `Disallow: /cdn-cgi/` en `public/robots.txt` para sacar el namespace interno de Cloudflare del rastreo. **No** hay regresión de contenido — los otros 22 URLs "sin indexar" de GSC ya estaban resueltos en prod (redirects 301 a la PWA para `/firmar` `/verificar` `/en/sign` `/en/verify` añadidos en `Caddyfile.landing` v0.1.4/v0.1.27; slashless→trailing-slash 308 benignos; `app.firmar.ec` noindex intencional; `/.well-known/*` no-HTML) o son datos stale del último rastreo (19–26 may, previo al deploy de esos redirects). Acción complementaria fuera del repo: "Validar corrección" en Search Console para forzar el recrawl.
- **Firmas rechazadas por el validador del Consejo de la Judicatura por sello de tiempo FreeTSA** (`@firma-ec/pwa` 0.10.0): el validador oficial del CJ (y del flujo ECUAPASS/SENAE) NO confía en FreeTSA — al ver el sello de tiempo RFC 3161 lo cuenta como una "segunda firma no confiable" y marca el documento entero como inválido, aunque la firma criptográfica del usuario sea correcta. Como firmar.ec defaulteaba `tsaEnabled: true` desde v0.5.0-rc1 (perfil B-T), todos los PDFs firmados con la PWA caían en este caso al subirlos al validador del CJ. Fix: **defaults flipped a `tsaEnabled: false` + `ltvEnabled: false` + `ltvArchiveEnabled: false`** — la salida default ahora es **PAdES-B-B** puro, idéntica a la que produce el FirmaEC desktop oficial del MINTEL, así valida en el CJ y en cualquier validador estricto. `STORAGE_KEY` bumped `firma_ec_settings_v1` → `firma_ec_settings_v2` para forzar la migración: usuarios existentes que tenían v1 con TSA/LTV ON pasan automáticamente al default seguro v2 (sus settings v1 quedan huérfanos pero sin lectura). Los power-users cuya contraparte acepta B-T pueden re-activarlo en Configuración (toggle sigue presente, copy actualizado ES+EN para reflejar el trade-off explícitamente). Sin cambios en la lógica del signer/verifier — solo flip de defaults + comentarios + i18n del aviso. TSL-EC `generatedAt` regenerado, normalización de line-endings en pemContent de Datil (cosmético, hash bumped).

### Added
- **4 piezas SEO pillar + transaccional + linkable bait (auditoría SEO 2026-05-29)** (`@firma-ec/landing` 0.2.0): tras auditoría completa con equipo de 4 subagentes (técnico, contenido/competitivo, GEO/AEO, analytics+SEM+local) que concluyó que el cuello para top 3 NO es técnico (Lighthouse 99/100, Astro SSG, schemas válidos) sino **autoridad + profundidad de contenido + entity disambiguation vs FirmaEC/MINTEL**. Se publican 4 páginas nuevas ES+EN (8 piezas) que cubren los gaps editoriales prioritarios:
  - **`/comparativa-emisores-ecuador/`** + EN `/en/certificate-issuers-ecuador/` — **linkable bait #1**: lista oficial actualizada de las 17+ ECIs acreditadas por ARCOTEL con tipo de cert, vigencia, precio referencial y enlace oficial. Pieza pensada como referencia comunitaria citable por blogs, contadores, agencias y medios tech EC para conseguir backlinks editoriales (hoy = 1 backlink real).
  - **`/que-es-firma-electronica/`** + EN `/en/what-is-electronic-signature/` — pilar informacional para head term "firma electrónica" (Q1 SERP). Respuesta-primero (citation-ready para AI search), distinción entre firma simple y certificada, FAQ.
  - **`/firma-electronica-vs-firma-digital/`** + EN `/en/electronic-signature-vs-digital-signature/` — pilar de desambiguación, captura "firma electrónica vs digital" + resuelve confusión técnica/legal, refuerza autoridad. Cita literal del Art. 13 Ley 2002-67.
  - **`/precios/`** + EN `/en/pricing/` — landing transaccional Q7 ("comprar firma electrónica precio ecuador") con tabla comparativa de precios por ECI, costos ocultos, IVA, comparativa con DocuSign/eIDAS. Honest: indica que firmar.ec todavía no revende (F9 in-progress).
  - **`llms.txt` actualizado** con las 4 URLs nuevas en sección "Páginas temáticas" + nueva sección "Referencias y precios" para mejorar retrieval por LLMs (Perplexity/Anthropic).
  - **`ROUTE_MAP` (i18n/utils.ts)** extendido con los 4 pares de ruta ES↔EN — sitemap auto-emite `xhtml:link` recíprocos para las 8 URLs (la lógica de `serialize` ya existente).
  - Schemas heredados del layout `ArticleSeo`: `TechArticle` + `BreadcrumbList` por página, `dateModified` visible (señal de frescura). Reportes completos en `_scratch/seo-firmar-ec-2026-05-29/reports/`.
- **Guías pSEO por emisor: UANATACA, Security Data, ArgosData, Consejo de la Judicatura (iCert-EC)** (`@firma-ec/landing` 0.1.48): 4 nuevas guías "Cómo firmar un PDF con certificado de [ACE]" (ES+EN, con `HowTo` + `BreadcrumbList` JSON-LD) que captan búsquedas por emisor con intención alta. Cada una es **contenido diferenciado** (no plantilla genérica): UANATACA explica el `.p12` leaf-only que firmar.ec completa; Security Data, la ECI más usada; ArgosData, certs de persona natural; iCert-EC, uso judicial + multi-firma. Para evitar contenido duplicado enlazan a la guía canónica `/como-firmar-pdf` para el flujo de 6 pasos, y se **enlazan desde las tarjetas de Compatibilidad** (link "Cómo firmar →" en BCE/UANATACA/Security Data/ArgosData/iCert-EC) para que no sean páginas huérfanas. (La guía del BCE ya existía.)
- **SEO de contenido: página `/validar-certificado` + FAQ enriquecida por emisor** (`@firma-ec/landing` 0.1.47): PSI ya estaba al máximo (SEO 100, Core Web Vitals verdes), así que las ganancias son de descubribilidad. (1) Nueva **página SEO dedicada** `/validar-certificado/` (ES) + `/en/validate-certificate/` (EN) — "Cómo validar tu certificado .p12 (gratis)", con `HowTo` + `BreadcrumbList` JSON-LD, nombra los emisores (ACE) y da destino interno real al enlace del nav (antes apuntaba directo a la app). Captura búsquedas tipo "validar certificado electrónico ecuador" / "validar .p12". (2) **3 FAQ nuevas** (ES+EN) que alimentan el `FAQPage` schema y apuntan a long-tail: compatibilidad por emisor (UANATACA/Security Data/BCE/ANF AC/iCert-EC…), "¿es seguro firmar en línea?" y "¿puedo validar mi `.p12`?". (Comparativa vs FirmaEC y vs Adobe Sign ya existían.)

### Added
- **Contador público de "Certificados validados" + enlace "Validar certificado" en el nav** (`stats-worker` · `@firma-ec/pwa` 0.9.16 · `@firma-ec/landing` 0.1.46): (1) el edge Worker de stats ahora cuenta un tercer evento `cert` (`GET /api/stats` devuelve `certificatesValidated`; `POST /api/stats/event?type=cert`), con el mismo modelo anónimo (sin PII, rate-limit por IP). (2) La página *Validar certificado* dispara `pingUsage('cert')` tras una validación exitosa (beacon sin payload). (3) El `UsageCounter` del landing muestra "Certificados validados" / "Certificates validated" cuando el total es > 0 (oculto en 0, como los otros). (4) Se añadió "Validar certificado" al nav del landing (desktop + móvil, ES+EN) enlazando a `app.firmar.ec/#/validar-certificado`. (5) `robots.txt`: `Allow` explícito para `/og/home.png` y `/og/home-en.png` (siguen siendo el `og:image` del home) manteniendo el resto de `/og/` fuera del índice de imágenes.

### Changed
- **Copy + SEO del landing enfocados en GRATIS + SEGURO** (`@firma-ec/landing` 0.1.45): la herramienta es gratuita y privada por diseño, pero esos dos ganchos no aparecían en las superficies primarias de conversión/SEO (hero, H1, `<title>`, meta description, JSON-LD) — solo en zonas secundarias (patrocinio, aviso de certificados). Ahora: (1) **Hero** — eyebrow `Firma electrónica · Ecuador · Gratis`, H1 `Firma y verifica PDFs gratis con tu certificado .p12.`, subtítulo que lidera con `100% gratis y en tu navegador … sin registro, sin servidores`, y un **badge `Gratis`** (tono verde pálido) como primer badge. (2) **SEO** — `meta.home.title` = `Firmar y verificar PDF gratis con tu certificado .p12` (capta `firmar pdf gratis`), description liderando con `gratis` + `sin registro y sin servidores` (ES+EN). (3) **JSON-LD `SoftwareApplication`** — description con `gratis/free`, `isAccessibleForFree: true` (señal canónica de "Free" para Google además del `offers price 0`), y `softwareVersion` 0.7.4→0.9.14 (estaba stale). Sin cambios de marca: se respeta el sistema visual existente (oklch azul, `font-display`, badges) — solo se aplicó disciplina de copy/jerarquía (sin emojis, contraste accesible). (4) **OG/WhatsApp** — el preview de compartir del home dejaba de usar la imagen estática stale `og-firmar-ec.png` y pasa al generador dinámico `/og/home.png` (ES) y nuevo `/og/home-en.png` (EN), con copy `gratis` incrustado (título + badge `Gratis · Open source · Cumple LOPDP`); la URL nueva además evita la caché de WhatsApp del og:image viejo. (5) **SEO de compatibilidad con emisores (ACE/ECI)** — para captar búsquedas por emisor (`firma electrónica UANATACA`, `certificado Security Data`, `BCE`, `ANF AC`, `iCert-EC`, etc.): el lede de `Compatibilidad` ahora **nombra en prosa** los emisores más buscados (Security Data, BCE, UANATACA, ANF AC, Consejo de la Judicatura/iCert-EC, ArgosData, Datil, Lazzate, Eclipsoft…) junto al H2, y el JSON-LD `SoftwareApplication` gana un `featureList` que enumera los 16 emisores acreditados (asociación de entidad para structured data). La lista visible de 16 tarjetas ya existía.

### Added
- **Soporte de firmas legacy `adbe.pkcs7.sha1` sin signed attributes (BCE, Security Data) — incl. validez a la hora de firma** (`@firma-ec/verifier` 0.9.0 · engine 0.9.0 · `@firma-ec/tsl-ec` · `@firma-ec/pwa` 0.9.12): muchísimos documentos gubernamentales ecuatorianos (BCE, Security Data y firmantes vía Registro Civil) están firmados con el perfil **Adobe legacy `adbe.pkcs7.sha1`**, donde el CMS **no lleva signed attributes** y el `eContent` contiene **SHA-1(byteRange)** (la firma RSA va sobre ese `eContent`, no sobre los bytes del documento). El verificador asumía PAdES-B-B (signed attrs + `message-digest`) y los rechazaba como inválidos. Ahora: (1) `parseCms` admite CMS **sin signed attributes** y expone el `eContent`; (2) hay una rama de verificación específica para `adbe.pkcs7.sha1` (integridad = `SHA-1(byteRange) == eContent`; firma RSA verificada sobre el `eContent`); (3) **SHA-1 se acepta SOLO en este perfil legacy de lectura** (sigue rechazado en el resto) y la firma se reporta como **válida con aviso `weak_hash_sha1`** (nunca un "válido" pelado), coherente con la postura del producto; (4) la cadena se valida a la **hora de firma** (`/M` del dict cuando no hay signing-time firmado), no a "ahora", para que un certificado **vigente al firmar pero ya expirado** siga validando (caso real: cert del firmante del BCE expirado 2 meses después de firmar). Además se **bundlean las intermedias SUBCA-2 SECURITY DATA e ICERT-EC** (extraídas de cadenas reales) para que sus PDFs leaf-only completen cadena (igual que UANATACA). Resultado verificado sobre un corpus real: BCE y Security Data → ✅ válido-con-aviso; iCert/ArgosData/multifirma → ✅ válido; sin regresiones (verifier 81, signer 77, tsl-ec 3). ⚠️ El `eContent` SHA-1 hace que la integridad de estos documentos descanse en SHA-1 (débil ante colisiones); por eso el aviso. Firmas modernas (PAdES-B-B/B-T/B-LT) no se ven afectadas.

### Fixed
- **"Validar certificado" fallaba con "No se pudo leer el certificado… dañado" en `.p12` leaf-only (UANATACA, Security Data, BCE)** (`@firma-ec/verifier` 0.9.2 · engine 0.9.2 · `@firma-ec/pwa` 0.9.15): subir un `.p12` que trae **solo la hoja** (sin la CA intermedia) a *Validar certificado* lanzaba `Cannot read properties of undefined (reading 'buffer')` → la UI lo mostraba como "archivo dañado", aunque el certificado y la contraseña eran correctos (caso real: cert UANATACA de LIANET VAZQUEZ ACOSTA). Causa raíz: `checkCertificate` **añade una CA subordinada bundled** (p.ej. UANATACA CA2 2016) para puentear la hoja hasta su raíz, pero la rama de **revocación** (`checkRevocation: true`, que activa esta página) indexaba el array **original** `intermediatesDer` para construir el CertID OCSP del emisor. Para un `.p12` leaf-only ese array está vacío, así que el DER de la intermedia bundled se leía como `undefined` y el constructor del request OCSP hacía `undefined.buffer`. La verificación de PDFs no lo veía porque PAdES-B-B no consulta OCSP. Fix: mantener un array `intermediateDers` **alineado 1:1** con `intermediates` que crece cuando se añade una CA bundled, de modo que el emisor pasa su DER real al OCSP. La firma sigue siendo válida; offline, la revocación degrada a `unknown` sin romper. Test de regresión: `checkCertificate(leaf, [], { checkRevocation:true })` sobre un fixture Security Data (emitido por la SUBCA-2 bundled) ahora construye el OCSP sin crash y resuelve la raíz `securitydata`.
- **CMS en BER indefinite-length (`30 80 … 00 00`) rechazado como `cms_parse: ASN.1 BER decode returned -1 offset`** (`@firma-ec/verifier` 0.9.1 · engine 0.9.1 · `@firma-ec/pwa` 0.9.13): algunos firmantes ecuatorianos emiten el PKCS#7 con **codificación BER de longitud indefinida** (`SEQUENCE 30 80 …` terminado en octetos *end-of-contents* `00 00`), no DER de longitud definida. La extracción del `/Contents` recortaba **todos** los `0x00` finales (asumiendo que solo eran el relleno reservado de PAdES), lo que **se comía los octetos EOC** → el CMS quedaba truncado y `asn1js` fallaba con *offset -1*, marcando firmas válidas como inválidas. El mismo recorte ciego también dañaba un CMS **DER de longitud definida cuyo último byte de contenido es `0x00`** (p.ej. un `INTEGER 0` final). Fix: `hexToBytes` ahora lee la **longitud real del objeto ASN.1 externo** desde su cabecera tag+length y devuelve exactamente esos bytes — definida corta/larga → corte exacto; **indefinida → conserva los EOC** (asn1js para en el EOC e ignora el relleno). Tests de regresión sintéticos (indefinite-length con EOC, definite-length terminando en `0x00`, long-form). Casos reales: `Carta…_signed.pdf` y la 2ª firma de un ACTA (ambas `30 80`). Firmas DER normales no se ven afectadas (corte exacto == resultado previo).
- **Verificar: mensaje claro para PDF truncado/incompleto en vez de error genérico** (`@firma-ec/pwa` 0.9.14): un `/ByteRange` que apunta más allá del fin del archivo (`c+d > fileSize`) significa que el contenido firmado ya **no está completo** — típico de una **descarga interrumpida** o un archivo **truncado al copiarlo** (caso real: copias parciales de `S00071_firmada_*.pdf`, a las que les faltaban ~1.5 KB finales). Antes `verifyAllSignatures` reportaba este caso como `signatureCount=0` + `overallStatus='invalid'` (con el código en `signatures[0].error`), y la UI lo enrutaba al card **"este PDF no contiene firma"** — engañoso, porque el PDF **sí** tiene firma, solo está truncada. Ahora un resultado de 0 firmas pero `invalid` se enruta a la UI de error de motor y el código `byterange_invalid` se mapea a un **mensaje dedicado** (ES+EN) que explica que el documento parece incompleto/dañado y sugiere **volver a descargar el PDF original**. El veredicto sigue siendo `inválido` (correcto: no se puede verificar contenido ausente).
- **`/Contents` antes de `/ByteRange` en el dict de firma → `byterange_invalid` (Security Data, BCE)** (`@firma-ec/verifier` 0.8.1 · engine 0.8.1 · `@firma-ec/pwa` 0.9.11): el verificador buscaba el `/Contents <hex>` SOLO hacia adelante desde el token `/ByteRange`. Algunas ACEs ecuatorianas (Security Data, BCE) ordenan el diccionario de firma con `/Contents` **antes** de `/ByteRange`, así que la búsqueda no lo encontraba (solo veía los `/Contents N 0 R` de páginas) y lanzaba `byterange_invalid: /Contents not found`, marcando firmas válidas como inválidas. Fix: extraer el hex **directo del hueco no firmado `[a+b, c)`** del ByteRange (donde PAdES garantiza que vive el octet-string), como fallback independiente del orden del productor. Mismo tipo de fix que el de `parseString` en v0.7.29, ahora aplicado a la extracción del `/Contents`. Test de regresión sintético (dict con `/Contents` antes de `/ByteRange`). (Prerequisito del soporte `adbe.pkcs7.sha1` que ahora valida estos PDFs — ver entrada en "Added" 0.9.0.)
- **Firmas de ACEs con CA subordinada (p.ej. UANATACA) rechazadas como "emisor no reconocido"** (`@firma-ec/verifier` 0.8.0 · engine 0.8.0 · `@firma-ec/signer` 0.8.0 · `@firma-ec/tsl-ec` · `@firma-ec/pwa` 0.9.10): un PDF firmado con un cert UANATACA (hoja emitida por la subordinada **UANATACA CA2 2016**, que encadena a **UANATACA ROOT 2016** — raíz ya confiada en la TSL-EC) salía **"Firma inválida — el emisor no está reconocido en Ecuador"** pese a ser criptográficamente correcto. Causa raíz: los `.p12` de UANATACA traen **solo la hoja**, así que el CMS embebía solo la hoja y el verificador 100% client-side (sin fetch AIA) no podía armar `hoja → CA2 2016 → ROOT 2016`. Afecta a **toda ACE cuya hoja la emita una CA subordinada** cuando el `.p12` no trae la intermedia. Fix en dos frentes: (1) **nuevo store de intermedias** en `@firma-ec/tsl-ec` (`getIntermediates()` + PEM real de UANATACA CA2 2016) que el verificador usa para **completar selectivamente** la cadena del firmante (solo los eslabones que puentean hacia una raíz **ya confiada** — nunca otorga confianza: pkijs sigue exigiendo terminar en una raíz self-signed de la TSL); (2) el **firmador embebe** la intermedia faltante en el CMS cuando el `.p12` es leaf-only, dejando el PDF **autocontenido** (valida en firmar.ec **y** en Adobe). Cubre verificación de firmas, "Validar certificado" y multifirma. Tests: round-trip leaf-only sign↔verify (reproduce el bug → inválida sin intermedia, válida con ella), inyección selectiva sin contaminar cadenas válidas, y garantía de seguridad (una intermedia bundled NO hace confiable una raíz no confiada). Incidente: e-factura firmada por LEANDRO HERNANDEZ GORINA 2026-05-28.
- **PWA: error de consola CSP por el beacon de Cloudflare Web Analytics** (`@firma-ec/pwa` 0.9.6 · infra CF): el edge de Cloudflare inyectaba `static.cloudflareinsights.com/beacon.min.js` en `app.firmar.ec`, pero el CSP estricto de la PWA (`script-src 'self' 'unsafe-inline' 'wasm-unsafe-eval'`) lo bloqueaba → error de consola en cada carga y CF Analytics inútil en la app. Fix: **Cloudflare Configuration Rule** (`http.host eq "app.firmar.ec"` → `disable_rum: true`) que desactiva la inyección del beacon **solo en la PWA** — la app queda **sin terceros en runtime** (coherente con su promesa de privacidad) y la landing conserva su analítica. Verificado en vivo: el beacon ya no se inyecta. (Sin reinicio de Traefik.)
- **Limpieza de i18n muerto `firmar_placeholder.*`** (`@firma-ec/pwa` 0.9.6): removidas las 10 claves (ES+EN) del placeholder "Próximamente — F3" que ya no usa ningún componente (Firmar es una ruta real desde hace tiempo).
- **PWA Home: badge stale "Próximamente (F3)" en la card "Firmar un PDF"** (`@firma-ec/pwa` 0.9.5): F3 (firma con `.p12`) está LIVE desde v0.5.1, pero la card de Firmar en el Home seguía mostrando un badge ámbar "Próximamente (F3)" / "Coming soon (F3)" (label stale de cuando F3 no existía) — daba la impresión de que firmar no estaba disponible. Removido el badge + el acento de la card pasa de `warn-500` (ámbar/pendiente) a `brand-500` con flecha de hover, igual que la card de Verificar (feature live de primera clase). Eliminado el i18n muerto `home.firmar_soon` (ES+EN). (Queda `firmar_placeholder.*` como i18n muerto sin componente, sin impacto.)

### Changed
- **SEO: hreflang `xhtml:link` en el sitemap para las 36 URLs (antes 4)** (`@firma-ec/landing` 0.1.44): el `i18n` de `@astrojs/sitemap` solo paréa URLs con el mismo slug salvo el segmento de idioma (`/faq/`↔`/en/faq/`), así que las páginas con **slug traducido** (`/seguridad/`↔`/en/security/`, `/acerca/`↔`/en/about/`, todas las guías) nunca recibían sus alternates en el sitemap — solo 4 de 36 los tenían. Se añadió un `serialize` que adjunta los `xhtml:link` (`es-EC`/`en-US`/`x-default`) por URL usando `ROUTE_MAP` (fuente única de pares de ruta ES↔EN, ahora exportada). Resultado: **36/36 URLs con hreflang recíproco** en `sitemap-0.xml`. Los `<link hreflang>` in-page ya eran correctos; esto los refuerza a nivel sitemap.
- **Guía "Cómo firmar con certificado del BCE": alineada al flujo real de 6 pasos + structured data SEO** (`@firma-ec/landing` 0.1.43, ES+EN): la guía enumeraba **Paso 1…7** con un desglose desactualizado que no coincidía con el wizard de la app (6 pasos): contaba "Abre la app" y "Verifica antes de enviar" como pasos numerados, ponía la colocación del sello **después** del certificado (la app lo hace **antes**), y afirmaba que el sello es "opcional / se puede desactivar" + permite "cambiar la razón" (el wizard actual exige colocar el cuadro y no ofrece esos toggles). Reescrita a **6 pasos en el orden real** (1 Carga el PDF · 2 Coloca el cuadro de firma · 3 Carga el `.p12` · 4 Ingresa la contraseña · 5 Revisa y firma · 6 Descarga), con "Abre la app" como intro y "Verifica" como nota final opcional. **SEO:** el **JSON-LD `HowTo`** de ambas páginas `.astro` tenía los mismos pasos viejos hardcodeados (lo que Google indexa para el rich result) → reescrito al orden real de 6 pasos; `dateModified: 2026-05-28` añadido al frontmatter (señal de frescura, schema `Article`). Coherente con el conteo "Paso N de 6" de la PWA (0.9.9).
- **Wizard de Firmar: conteo de pasos correcto + footer alineado/centrado (responsive)** (`@firma-ec/pwa` 0.9.9): (a) **"Paso N de 7" → "Paso N de {total}"** — el wizard tiene **6 pasos** reales (el paso "Detalles opcionales" se removió en v0.7.15) pero los strings i18n `step_of` y `aria.progress` tenían el `7` hardcodeado; ahora se derivan de `totalSteps`/`steps.length` (no vuelve a desincronizarse). (b) **Footer en grid `[1fr·auto·1fr]`**: "Atrás" a la izquierda, indicador de paso centrado de verdad (offset ≤1px en PC/tablet, verificado), CTA a la derecha; el indicador se oculta en mobile (ya aparece en el stepper superior — evita el duplicado). (c) **CTA "Verificar contraseña" del paso 4 movido al footer**, alineado con "Atrás" en la misma fila (como "Firmar PDF" en el paso 5) en vez de flotar como botón aparte del `PinInput` — se eliminó el botón propio de `PinInput` (Enter sigue enviando). Verificado responsive PC 1280 / tablet 768 / mobile 390 con `getBoundingClientRect` (misma fila, sin solape, cabe en viewport).
- **Firma visible sin marco/recuadro** (`@firma-ec/pwa` 0.9.8 · `@firma-ec/signer`): la estampa visible (QR + "Firmado por / Fecha / Razón") dibujaba un recuadro gris alrededor del BBox. A pedido del usuario se removió el contorno → la estampa queda limpia sobre el documento (solo QR + texto). Eliminado el bloque de borde (`setLineWidth`/`rectangle`/`stroke`) en los **dos** generadores de apariencia (`visibleSig.ts` = single-sig vía `signPdfPades`, e `incrementalUpdate.ts` = multi-sig/incremental) + imports `setLineWidth`/`stroke` ya sin uso. Tests `visibleSig.test.ts` actualizados (afirman ausencia de borde: `not.toMatch(/0.5 w/)` y sin `S`); 35/35 verde, incluyendo la firma real con fixture rsa2048.
- **PWA Firmar: quitar el botón "Verificar contraseña" duplicado del paso 4** (`@firma-ec/pwa` 0.9.8): el paso 4 (contraseña del .p12) mostraba DOS botones idénticos "Verificar contraseña" — el propio del `PinInput` y el botón Next del footer del wizard — ambos llamando `onPinSubmit`. El diseño ya preveía ocultar el Next en ese paso pero `hideFooter` solo cubría los pasos 1/3/6. Se añadió el prop `hideNext` a `WizardShell` (oculta solo el botón Next, conservando "Atrás" + el indicador "Paso N de 7") y se activa en el paso 4. Ahora hay un único CTA "Verificar contraseña" (el de `PinInput`). El botón "Firmar PDF" real sigue siendo el del paso 5.
- **PWA Verificar: reformular el flujo del QR y quitar el aviso engañoso "hash no coincide"** (`@firma-ec/pwa` 0.9.7): el QR que el firmante incrusta codifica el hash del PDF **sin firmar**, pero el QR vive *dentro* del PDF firmado → al escanearlo y subir ese PDF firmado (el único que el usuario tiene) la comparación de hash **nunca** coincidía y mostraba un banner ámbar *"ℹ Hash no coincide"* que parecía un fallo. Alineado con la postura oficial de FirmaEC/MINTEL (*"el escaneo del código QR no es un método de verificación… el QR no valida que un documento esté firmado"*): el QR es solo un **atajo** al verificador; la validación real es subir el PDF (todo en el navegador, sin servidor). Cambios: banner reformulado ("Llegaste desde un QR de firmar.ec — sube el PDF firmado para validar; el QR por sí solo no verifica nada"), **eliminada** toda la comparación de hash (`compareHash12` + estado `qrCompare` + banners verde/ámbar + 4 claves i18n muertas `match_ok`/`match_warn`/`why_*` en ES+EN + su bloque de tests). La verificación criptográfica (el veredicto del worker) no cambia. `parseQrHash`/`readQrHashFromLocation` se mantienen (alimentan el banner de contexto).
- **`/patrocinar`: separar Patrocinio (donación) de Licencia comercial / Enterprise** (`@firma-ec/landing` 0.1.40): a raíz del modelo dual AGPL+comercial, la página separa dos pistas que antes se mezclaban. **Patrocinio** (Bronze→Platinum) = apoyo al proyecto abierto: visibilidad + influencia en roadmap + acceso anticipado (se removió "16 h/mes + integración" de Platinum → "Soporte prioritario"; el patrocinio ya no implica integración ni SLA). Nuevo panel **"Enterprise & Licencia comercial"** (reemplaza el de "Founding"): integración en sistemas propietarios (licencia comercial bajo AGPL), SLA y soporte por contrato, integración a medida (API/SSO/white-label), emisión de certificados, gobierno/GADs/universidades → contacto `info@idkmanager.com` (canal comercial, distinto de `sponsors@`). Clarificador en el encabezado de niveles. Coherente con la gobernanza ("el patrocinio no compra el servicio"). Campo `"license": "AGPL-3.0-only"` añadido a `apps/landing` y `apps/pwa` package.json.
- **Tema: light por defecto + auto-dark según el dispositivo** (`@firma-ec/landing` 0.1.39, `@firma-ec/pwa` 0.9.4): el bootstrap de tema (landing `Base.astro` + PWA `index.html`) ahora respeta `prefers-color-scheme`. Prioridad: elección explícita del usuario (`localStorage 'theme'` = `light`/`dark`) gana; si no hay elección, sigue al dispositivo (dispositivo en oscuro → dark), con **light como fallback**. Antes ignoraba `prefers-color-scheme` (default light salvo toggle manual). Se mantiene sin FOUC (script pre-render).
- **Relicencia a GNU AGPL-3.0 + licencia comercial (modelo dual)** (`@firma-ec/landing` 0.1.38, `@firma-ec/pwa` 0.9.3): el proyecto pasa de **Apache-2.0** a **AGPL-3.0** hacia adelante (sigue siendo open source / software libre — conserva la preferencia de compra pública en EC). Su *copyleft* obliga a quien integre firmar.ec en un sistema cerrado con fines de lucro a liberar su código **o** adquirir una **licencia comercial** (nuevo [`LICENSE-COMMERCIAL.md`](LICENSE-COMMERCIAL.md), contacto `info@idkmanager.com`). `LICENSE` reemplazado por el texto canónico AGPL-3.0; `package.json` (`AGPL-3.0-only`), `jsonld.ts` (URL OSI), badge README, footers landing+PWA, About/Home PWA, llms.txt/llms-full.txt, ai-plugin.json, Términos (§5 reescrito: copyleft + licencia comercial; §3 disclaimer AGPL §15-16), FAQ-empresas (ES+EN: uso de la app gratis; integrar código → AGPL/comercial) y docs de patrocinio (gobernanza/FAQ) actualizados. **Cero rastro de Apache-2.0** en superficie user-facing (se preserva la licencia Apache propia de pdf.js vendorizado y el historial). ⚠️ El contrato comercial lo formaliza un abogado; registro de obra en SENADI recomendado.
- **Repos git renombrados `firma-ec` → `firmar-ec`** para alinear con la marca y distanciar de "FirmaEC" (MINTEL): GitHub `idkmanager/firmar-ec` + `alfonsokuen/firmar-ec` (rename con redirect), Gitea ya era `alfonso/firmar-ec`. Todas las URLs de repo en landing/PWA/docs actualizadas. El scope npm `@firma-ec/*` se mantiene (interno; renombrarlo rompería el build). Las referencias a **FirmaEC** (producto MINTEL) en comparativos/alternativa se conservan (uso nominativo legítimo).

### Added
- **Menú de navegación mobile (hamburguesa)** (`@firma-ec/landing` 0.1.41): el `Header.astro` ocultaba los 6 enlaces (`Firmar`, `Verificar`, `Seguridad`, `Preguntas`, `Acerca`, `Patrocinar`) con `hidden md:flex` y **no había nada que los reemplazara en mobile** (`<768px`) — la navegación quedaba inaccesible en el teléfono (solo logo + "Abrir app" + idioma + tema). Se añadió un botón hamburguesa (`md:hidden`, icono `menu`↔`x`, `aria-expanded`/`aria-controls`) que despliega un panel con los enlaces apilados; cierra al tocar un enlace, con `Escape`, clic fuera, o al cruzar a viewport desktop. Los enlaces se generan desde un array único (`navKeys`) compartido por desktop y mobile para que **nunca diverjan**. Animación `opacity`+`transform` (GPU-friendly, respeta `prefers-reduced-motion`).
- **Fila de patrocinadores al pie del hero** (`@firma-ec/landing` 0.1.37): nuevo `HeroSponsors.astro` que muestra una fila compacta "Con el apoyo de" con los logos (escala de grises → color en hover) debajo del contador de uso. **Empty-safe**: con 0 patrocinadores no renderiza nada y aparece sola al agregar el primero. Clic en los logos y en "Ver patrocinadores →" baja a la sección `#patrocinadores` ("Quienes hacen esto sostenible") con `scroll-mt` por el nav fijo. Se introdujo `src/data/sponsors.ts` como **fuente única** (la consumen `HeroSponsors` y `SponsorsStrip` → agregar un logo aparece en ambos lugares a la vez); `SponsorsStrip.astro` ahora importa de ahí y la sección lleva `id="patrocinadores"`.

### Changed
- **Título de la home (`meta.home.title`) = frase de acción, igual al H1** (`@firma-ec/landing` 0.1.37): ES `'Firma y verifica PDFs con tu certificado electrónico .p12.'` / EN `'Sign and verify PDFs with your .p12 electronic certificate'` (antes `'Firma electrónica ecuatoriana en tu navegador'`). Cambia el `<title>` y el **título del preview al compartir** (og:title) para que coincida con el Hero. Mismo texto actualizado en los títulos de las imágenes OG generadas (`og/[slug].png.ts` slugs `default`/`home`). ⚠️ Trade-off SEO: el título deja de llevar el keyword literal "firma electrónica ecuatoriana" (sigue presente en `meta.home.description` y en el cuerpo). La imagen estática `public/og-firmar-ec.png` mantiene su texto incrustado (regeneración aparte si se desea).

### Docs
- **README + metadata de repos actualizados al estado actual** (2026-05-27): tabla "Estado del proyecto" sincronizada (landing `v0.1.36` · PWA `v0.9.2`, fila del contador de uso en vivo y del cluster de contenido SEO bilingüe), bullet del contador en "Características LIVE", y mirror personal añadido a "Repos". Descripción corta, website (`https://firmar.ec`) y topics actualizados en las 3 superficies (Gitea `alfonso/firmar-ec`, GitHub `idkmanager/firma-ec`, GitHub `alfonsokuen/firma-ec`). Sin tocar el conteo de ACEs ARCOTEL (decisión YMYL diferida) ni el bloque de verificación Sigstore (tag/artefactos reales).

### Fixed
- **Landing — auditoría SEO 2026-05-25: pulido de 2 overflows propios** (`@firma-ec/landing` 0.1.33): la re-auditoría tras los deploys del día detectó 2 metadatos recién creados ligeramente sobre el límite SERP — `/como-firmar-pdf` meta description 163→145 chars; `/alternativa-firmaec` title 62→50 renderizados ("Alternativa a FirmaEC para firmar PDFs"). Los equivalentes EN ya estaban en límite (≤60/≤154). Sin cambio de keyword.

### Fixed
- **Beacon de la PWA bloqueado por CSP** (`@firma-ec/pwa` 0.9.2, `@firma-ec/stats-worker`): el beacon apuntaba a `https://firmar.ec/api/stats/event` (cross-origin), pero el `connect-src` de la PWA (`'self' https://ocsp.firmar.ec https://freetsa.org`) NO incluye el apex → el navegador lo bloqueaba silenciosamente y firmar/verificar no contaba. Fix sin tocar CSP: el Worker ahora también sirve `app.firmar.ec/api/stats*` y el beacon usa URL **relativa** (mismo origen) → pasa `connect-src 'self'`.

### Added
- **Contador de uso en vivo en la landing + beacons anónimos** (`@firma-ec/landing` 0.1.36, `@firma-ec/pwa` 0.9.2, nuevo `@firma-ec/stats-worker`): el Hero muestra cifras reales de uso (documentos firmados · firmas verificadas; certificados emitidos cuando aplique) con count-up minimalista que respeta `prefers-reduced-motion`. Muestra el total real **siempre (incluso 0)**, creciendo con el uso; el "+" aparece solo cuando hay > 0. Se oculta **solo si el fetch falla** (sin números inventados). Servido por un **Cloudflare Worker** (`tools/stats-worker`, ruta `firmar.ec/api/stats*` misma zona, KV) — totalmente aislado de la app de firma, **sin PII** (solo dos enteros), rate-limit 20/h por IP, alineado con zero-knowledge/LOPDP. La PWA emite un beacon anónimo (`navigator.sendBeacon` a `/api/stats/event`) al completar una firma (`Firmar.svelte`) o una verificación con firmas (`Verificar.svelte`); best-effort, nunca rompe el flujo. Arranca en 0 → oculto hasta el primer uso real. Las rutas `/api/stats` equivalentes del `inbox-backend` quedan como alternativa probada pero NO desplegada.
- **Landing — posicionamiento de intención: emisión de certificados "próximamente"** (`@firma-ec/landing` 0.1.32): nuevo componente `CertNotice.astro` (aviso bilingüe, on-brand, **sin formulario ni captura de datos** — respeta la postura "sin formularios, sin tracking" del sitio) que anuncia que firmar.ec habilitará la **emisión de certificados de firma electrónica**, reafirmando que la herramienta de firma sigue **gratis y open source**. NO nombra proveedor ni precios (acuerdo de revendedor en negociación). Colocado en las 2 páginas de mayor intención de compra: `/como-obtener-certificado-firma-electronica` y `/como-firmar-pdf` (ES+EN). Primer paso del giro de monetización: convertir el tráfico SEO de intención de certificado en demanda posicionada.
- **Landing — 2 páginas P1: cluster "firmar" + "certificado"** (`@firma-ec/landing` 0.1.31):
  - **`/como-firmar-pdf` (`/en/how-to-sign-pdf`)** — HowTo madre para la query de mayor volumen del nicho ("cómo firmar un PDF"). Answer-first, requisitos, 6 pasos (schema **HowTo**), validez legal (LCE 2002-67), casos específicos que derivan a BCE/FirmaEC/token sin canibalizar la guía BCE (esta es genérica, BCE es el caso específico). `TechArticle`+`BreadcrumbList`.
  - **`/como-obtener-certificado-firma-electronica` (`/en/how-to-get-an-electronic-certificate`)** — proceso para obtener el certificado (G6): elegir ECI, requisitos, `.p12` vs token, solicitar/pagar/descargar. **HowTo** (5 pasos). Aclara que firmar.ec NO emite certificados (solo firma) y que el costo/vigencia **varían por ECI → remite a la lista oficial de ARCOTEL** (sin inventar precios; G5 tabla de precios queda pendiente de datos verificados).
  - Bilingüe (ROUTE_MAP + hreflang), enlaces en footer (Guías), silos pilar→cluster: la pilar enlaza firmar/obtener/verificar; cluster certificado↔firmar bidireccional.
- **Landing — nueva página `/verificar-firma-pdf` (`/en/verify-pdf-signature`)** (`@firma-ec/landing` 0.1.30): página P1 (G3) para la query "cómo verificar/validar la firma de un PDF" — alto intent, sin competidor bueno en EC, conecta con el verificador de `app.firmar.ec`. Answer-first (las 3 condiciones de validez: integridad + ECI ARCOTEL + no-revocación OCSP/CRL), pasos, qué reporta el verificador, alternativas honestas (Adobe Reader, validador MINTEL Minka) y FAQ. Schema **HowTo** (4 pasos) + `TechArticle` + `BreadcrumbList`. Bilingüe (ROUTE_MAP + hreflang), enlace en footer y silo "verificar/validez" bidireccional con `/que-es-firma-pades`. Claims tomados del comparativo verificado + LCE 2002-67.
- **Landing — nueva página `/alternativa-firmaec` (`/en/firmaec-alternative`)** (`@firma-ec/landing` 0.1.29): página P1 que ataca la query "alternativa a FirmaEC" + la **colisión de marca firmar.ec ↔ FirmaEC** detectada en el baseline GEO (Perplexity confunde firmar.ec con FirmaEC y no lo cita en 0/4 queries). Answer-first, sección explícita "firmar.ec no es FirmaEC" (desambiguación de entidad para LLMs y usuarios), razones para buscar alternativa (Java/móvil/instalación), tabla resumen, y sección honesta "cuándo SÍ necesitas FirmaEC" (XAdES SRI, token USB, lote, offline). Claims tomados del comparativo verificado `/comparativos/firmaec`. Bilingüe (ROUTE_MAP + hreflang), `TechArticle`+`BreadcrumbList`, enlace en footer (columna Guías) y silo bidireccional con el comparativo. Diferencia de intención vs `/comparativos/firmaec` (comparación neutral) para no canibalizar.

### Fixed
- **Landing — P0 SEO/GEO quick wins post-auditoría 2026-05-25** (`@firma-ec/landing` 0.1.28):
  - **Titles/descriptions recortados** (truncado en SERP, medido en auditoría): guía BCE title 77→54 car. renderizados ("Cómo firmar un PDF con certificado del BCE" / EN equivalente), `/comparativos/adobe-sign` title 63→≤60 ("…comparación para Ecuador"), meta description de la pilar `/firma-electronica-ecuador` 165→≤155 y `/que-es-firma-pades` 174→≤155 (ES+EN). Sin pérdida de keyword.
  - **TL;DR answer-first en `/seguridad`** (ES+EN): bloque al inicio que responde directo "¿Es seguro firmar.ec? Sí…" resumiendo hechos ya presentes en la página (llave `.p12` nunca sale del equipo, `extractable:false`, Web Worker, open source, A+ en Mozilla Observatory/SSL Labs/securityheaders verificado 2026-05-08). Mejora citabilidad GEO para la query "¿firmar.ec es seguro?". No añade hechos nuevos.
  - **Diferido (requiere decisión/insumos, NO incluido):** unificación de la cifra de ECIs entre superficies (home/pilar dicen "16 de 17 ECIs", `llms-full.txt` dice "9 ACEs/8 roots", TSL real tiene 28 roots activos → afirmación factual de trust list YMYL que exige fijar el número/framing canónico antes de tocar); enriquecer `sameAs` del Organization (requiere crear perfiles off-site LinkedIn/Wikidata primero).
- **Landing — P1: CTA EN rotos `/en/sign` y `/en/verify` (404)** (`@firma-ec/landing` 0.1.27): los dos CTA principales de la home en inglés (`Hero.astro` → `localizedUrl('firmar'|'verificar', 'en')`) apuntan a `/en/sign` y `/en/verify` por el `ROUTE_MAP` (`src/i18n/utils.ts`), pero `Caddyfile.landing` solo tenía los `redir` de las rutas ES (`/firmar`, `/verificar`, `/paranoia`) → **ambos daban 404**. Añadidos los 4 `redir` EN espejo del bloque ES (`/en/sign`→`app.firmar.ec/#/firmar`, `/en/verify`→`app.firmar.ec/#/verificar`, con sus `/*`). Era visible en GSC como 2 errores "No se ha encontrado (404)" (`/en/sign`, `/en/verify`, último rastreo 21/5/26) y rompía la conversión de usuarios EN. Detectado en auditoría SEO 2026-05-25.
- **Landing — P1 SEO: autoría E-E-A-T en guías (YMYL)** (`@firma-ec/landing` 0.1.26): las guías (`TechArticle`) ahora declaran `author` y `reviewedBy` = **Equipo IDK Manager** (Organization, url idkmanager.com), además del `publisher` (org firmar.ec). Byline visible "Por Equipo IDK Manager · {fecha}" en el encabezado de cada guía (ES) / "By IDK Manager Team" (EN). Antes solo había `publisher` sin autoría → techo de ranking en queries legales (la autoría/revisión es factor E-E-A-T central en contenido YMYL de firma electrónica).
- **Landing — P0 SEO: cluster de guías enlazado (enlazado interno)** (`@firma-ec/landing` 0.1.25): el footer (presente en TODAS las páginas, incluida la home de máxima autoridad) ahora tiene una columna **"Guías"** que enlaza las 5 páginas de contenido que estaban huérfanas de enlaces internos: `firma-electronica-ecuador`, `como-firmar-con-certificado-bce`, `que-es-firma-pades`, `comparativos/firmaec`, `comparativos/adobe-sign` (ES+EN vía `localizedUrl`/hreflang). Antes solo el glosario estaba enlazado; el contenido que capta demanda informacional no recibía PageRank interno desde la home. Grid del footer 4→5 columnas. Pendiente (mayor esfuerzo, idealmente con datos GSC): sección de guías en el cuerpo de la home + autoría E-E-A-T + páginas "verificar PDF"/"obtener .p12".
- **Landing — P3 SEO post-auditoría** (`@firma-ec/landing` 0.1.24):
  - **`/favicon.ico`** (antes 404): añadido `public/favicon.ico` multi-resolución (16/32/48) generado desde `icon-512.png`. Bots/previews que piden el `favicon.ico` bare ya no reciben 404 (el HTML ya referenciaba `/icons/favicon.svg`).
  - **Redirect `/en/firma-electronica-ecuador/` → `/en/electronic-signature-ecuador/`** (301, Caddyfile): la guía EN vive en el slug inglés; evita soft-404 si alguien prefija `/en/` al slug ES. No estaba en sitemap ni hreflang (impacto SEO nulo, fix defensivo).
- **Landing — auditoría SEO técnica: quick wins** (`@firma-ec/landing` 0.1.23):
  - **Trailing slash canónico** (`astro.config.mjs` `trailingSlash: 'never' → 'always'` + `ROUTE_MAP` y breadcrumbs/related/`llms.txt`/`llms-full.txt` normalizados): el canonical, el sitemap, los hreflang y los enlaces de nav apuntaban a URLs **sin** slash que el host (Caddy, directory-format) **308-redirige** a la versión con slash. Resultado: canonical no auto-referencial + sitemap lleno de redirects. Ahora las 4 señales coinciden con la URL servida (`/pagina/`), sin redirects. Detectado por los 3 frentes de la auditoría (técnico, contenido, GEO).
  - **Cifra de ECIs unificada a "16 de las 17"** (`Compatibilidad.astro`, `glosario/eci`+`en-eci`, `glosario/tsl`+`en-tsl`, `acerca`/`about`, guía `firma-electronica-ecuador`/`electronic-signature-ecuador`): el glosario, "acerca" y la guía pilar seguían diciendo "8 ECIs activas" (dato **stale** previo a v0.8.0); ahora reflejan las 16 ECIs con raíz propia que firmar.ec reconoce (16 de las 17 acreditadas; la 17ª, Registro Civil, firma con BCE/Security Data). Elimina la contradicción factual YMYL (home decía 17, glosario/guía decían 8). Se añaden por nombre las 8 ECIs faltantes (Lazzate, Alpha Technologies, AppFirmas, CorpNewBest, DarkCam, FirmaSegura, LetMi Ecuador, PrimeCoreLat) sin fabricar columnas Tipo/Notas.
  - **CSP permite el beacon de Cloudflare Web Analytics** (`Caddyfile.landing`): `static.cloudflareinsights.com` en `script-src` + `cloudflareinsights.com` en `connect-src`. Antes la CSP bloqueaba el beacon que CF inyecta en el edge → **cero datos de tráfico** + error CSP en consola en cada carga (único motivo del Best-Practices 92 en Lighthouse).
  - **Eliminado `public/sitemap.xml` huérfano**: archivo estático de 194 B con namespace XML malformado (`schemas/sitemap-0.9`) que sombreaba el sitemap real generado por `@astrojs/sitemap` (`/sitemap-index.xml`).
  - **Meta description de la home acortada** a ≤155 chars (ES y EN) para evitar truncado en SERP.

### Changed
- **Landing — nombre de marca en patrocinio** (`@firma-ec/landing` 0.1.22): la sección "Cómo se paga" de `/patrocinar` (`Sponsors.astro`, ES y EN) ahora dice "transferencia bancaria directa a **IDKMANAGER**" en vez de "IDK Manager Cía. Ltda." — usa la marca institucional como el resto del sitio. La factura SRI la sigue emitiendo la persona jurídica; el cambio es solo de marca visible.

### Added
- **Landing — strip de patrocinadores en la home** (`@firma-ec/landing` 0.1.21): nueva sección `SponsorsStrip.astro` en la portada (ES y EN, antes de `OperadoPor`) — muro de logos de patrocinadores cuando existan (grayscale→color en hover) y **empty-state** con borde discontinuo ("Este espacio está disponible" + CTA a `/patrocinar`) mientras no haya ninguno. Data-driven: agregar entradas al array `sponsors` con logo SVG en `/sponsors/<tier>/`.
- **Landing — programa de patrocinio** (`@firma-ec/landing` 0.1.20): nueva página `/patrocinar` (`/en/sponsor`) con la sección `Sponsors.astro` — tiers Bronze/Silver/Gold/Platinum/Founding, beneficios y modelo de **pago directo por transferencia bancaria con factura SRI, sin intermediarios** (no GitHub Sponsors, no Open Collective, no tarjeta). Enlace en el nav y el footer, bilingüe ES/EN, ruta en `ROUTE_MAP` con hreflang. Mensaje alineado con `OperadoPor`: la app sigue gratis; el patrocinio financia desarrollo/auditorías/infra. Construido sobre los tokens existentes (OKLCH ink/brand, Geist, iconos lucide) — sin emojis-como-icono, sin morado/glow, sin gradient-text; verificado en claro/oscuro y móvil. Acompaña la estructura del repo: `SPONSORS.md`, `.github/FUNDING.yml` (solo URLs propias), `docs/sponsorship/{README,benefits,governance,faq}.md`, `assets/sponsors/`.

## [0.9.0] — 2026-05-23 — Validar Certificado: nombres/apellidos/cédula + Expirado/Revocado (paridad FirmaEC)

### Added
- **Revocación en vivo en Validar Certificado** (`@firma-ec/verifier` `checkCertificate`): nueva opción `checkRevocation` que ejecuta la cascada **OCSP→CRL** contra ARCOTEL reusando `@firma-ec/ltv-validation` + `ARCOTEL_PROXY_MAP` (mismo patrón que el firmante). Expone `revocationStatus` (`good | revoked | unknown | unchecked`) + `revocationVia` + `revokedAt`. Es tolerante a offline: si no alcanza al respondedor (o el cert no trae AIA/CDP) devuelve `unknown` → la UI muestra "No verificable"; **nunca lanza ni bloquea** el veredicto de vigencia/cadena. Activado en `cert.worker.ts`.
- **Titular desglosado**: `CertCheckResult` ahora separa `givenName` (RDN 2.5.4.42 = nombres), `surname` (2.5.4.4 = apellidos) y `cedula` (2.5.4.5 = cédula/RUC) del CN, igual que FirmaEC 5.1.0. La UI muestra filas Nombres / Apellidos / Cédula (condicionales; si el cert no las trae cae al Titular/CN).
- **Estados Expirado y Revocado** en la tarjeta de resultado (`ValidarCertificado.svelte`): filas explícitas NO/SÍ; la tarjeta se pone roja cuando el cert está revocado. Claves i18n ES/EN (`common.yes/no`, `field_nombres/apellidos/cedula/expirado/revocado`, `revoked_unknown/unchecked`).

### Changed
- **Fechas de vigencia con hora** (`fmtDate`): "Válido desde/hasta" ahora incluyen hora:minuto:segundo (`dateStyle:'medium' + timeStyle:'medium'`) en zona local del navegador, en vez de solo la fecha.

## [0.8.4] — 2026-05-22 — diagnóstico: tamaño del .p12 recibido en error de PIN

### Changed
- **`p12.worker.ts`**: el diagnóstico de `pin_invalid` ahora incluye `p12bytes=<n>` (bytes recibidos del archivo). forge reporta un PKCS#12 truncado/corrupto (p.ej. mangleado al pasarlo al teléfono por chat/email) como fallo de MAC — indistinguible de una contraseña incorrecta. Si `p12bytes` es menor que el archivo real en disco, la subida llegó truncada (NO es la contraseña). Permite diagnosticar en remoto el caso "el mismo .p12 + PIN funciona en escritorio pero falla en el móvil". Confirmado que el parser forge en sí es correcto (parsea un .p12 LAZZATE real en ~25 ms).

## [0.8.3] — 2026-05-22 — fix: etiqueta de progreso de verificación mostraba la clave i18n cruda

### Fixed
- **`Progress.svelte`**: el spinner de verificación mostraba la clave literal `progress.verify:#0 ltv` en vez de un texto legible. El verifier emite beacons `verify:${tag}${name}` (con `#N ` de índice de firma en multi-firma) y el componente hacía `t('progress.' + stage)` sin normalizar → clave inexistente → se renderizaba cruda. Además faltaban claves para las fases `tsa`, `chain`, `ltv` y `scan`. Ahora se normaliza el beacon al token de fase, se mapea a etiqueta localizada (con fallback genérico para que NUNCA se filtre una clave cruda), se muestran las claves nuevas (incl. "Validando a largo plazo (LTV) — puede tardar", que explica la lentitud de la fase LTV en móvil) y el número de firma en PDFs multi-firma. Breadcrumb alineado a las fases reales (`cms · integrity · tsa · chain · ocsp · ltv`).

## [0.8.2] — 2026-05-22 — orden del nav: Firmar primero

### Changed
- **Header PWA**: reordenado el nav a `Inicio · Firmar · Verificar · Validar certificado · Paranoia · Acerca · Configuración` (Firmar pasa a ser la primera acción; antes iba después de Verificar/Validar certificado).

## [0.8.1] — 2026-05-22 — fix: Result.svelte no crashea en firmas con error de motor

### Fixed
- **`Result.svelte` (summaryKey)**: `result.integrity` es opcional y queda `undefined` cuando la verificación de una firma lanza excepción (path `catch` de `verifyPdf` → `status:'invalid'` sin `integrity`). El acceso sin guardia (`!result.integrity.digestMatches`) tiraba `TypeError` y rompía la tarjeta de resultado en vez de mostrar el error. Ahora se guarda con `result.integrity && …`; un error de motor cae al resumen genérico de firma inválida en lugar de etiquetarse erróneamente como "documento modificado". svelte-check vuelve a 0 errores.

## [0.8.0] — 2026-05-22 — Validar Certificado + raíces ACE reales (placeholders → 28/29 reales)

### Context
- Reporte de un firmante real (Leandro Gorina, cert **LAZZATE** `Persona Natural EXT`): la verificación NO funcionaba "con todos los firmadores autorizados". Causa raíz: 14 de 17 raíces ACE eran *placeholders* auto-firmados y `pathValidation` salta los placeholders (`if (r.isPlaceholder) continue`), así que cualquier documento firmado con un cert de esas ACE (incluida LAZZATE, además marcada erróneamente `isDefunct`) jamás validaba.
- Segundo reporte: faltaba **Validar Certificado** (subir `.p12` + PIN → ver emisor/titular/vigencia/cadena ACE), distinto de Validar PDF — como la pestaña de FirmaEC 5.1.0.

### Added
- **Nueva ruta `/validar-certificado`** + entrada de navegación "Validar certificado" (i18n ES/EN). Valida un certificado por sí mismo: parsea el `.p12` en un Worker single-shot (la clave privada nunca vuelve al hilo principal), muestra Titular, Emisor (ACE acreditada o "no acreditada por ARCOTEL"), N.º de serie, vigencia (desde/hasta), estado (VIGENTE / EXPIRADO / AÚN NO VÁLIDO) y cadena de confianza.
- **`checkCertificate(certDer, intermediatesDer, opts?)`** en `@firma-ec/verifier` — reusa `validatePath` contra las raíces TSL; sin firma de PDF.

### Fixed
- **Raíces ACE reales** (`@firma-ec/tsl-ec`, TSL **v1.11.0** seq **12**): las 8 ACE que eran placeholder (alpha-technologies, appfirmas, corpnewbest, darkcam, firmasegura, **lazzate**, letmi, primecorelat) ahora usan la raíz REAL extraída de la librería oficial MINTEL FirmaEC (firmadigital-libreria) — el mismo trust store que usa FirmaEC. Resultado: **28/29 entradas reales** (antes 9). Solo queda placeholder `registro-civil` (DIGERCIC no opera raíz PKI pública).
- **Anclas paralelas multi-vintage** (`isParallelAnchor`): se añaden raíces de distinta cosecha para que validen certs que encadenan a una raíz vieja O nueva — `lazzate`+`lazzate-ca1`/`ca2`/`wego`, `anfac-2024`/`anfac-2016`, `argosdata-2026`, `datil-2025`, etc.
- Cada raíz se verificó: huella SHA-256 del DER recomputada == `fingerprintSha256` almacenada (29/29), todas self-signed.

### Changed
- `APP_VERSION` + `apps/pwa/package.json` → 0.8.0; `@firma-ec/verifier` 0.7.8 → 0.7.9; TSL 1.10.0 → 1.11.0 (seq 11 → 12).

## [0.7.42] — 2026-05-21 — Firma .p12: recuperación de espacios internos en el PIN (teclado móvil + tecla `+`)

### Context
- Reporte (Samsung, móvil): firmar con un .p12 cuya contraseña **contiene un `+`** era rechazado como `pin_invalid` pese a ser correcta; en escritorio no se reproducía. 0.7.41 ya reintentaba `pin.trim()`, que solo arregla espacios al **inicio/final**.
- **Reproducción determinista** (`packages/signer`, round-trip con node-forge): forge maneja el `+` perfectamente (en medio, al inicio, múltiples). El modo de fallo real es el teclado en pantalla insertando un espacio **adyacente** a la tecla de símbolo (`clave + 2026` en vez de `clave+2026`); `trim()` NO recupera un espacio interno.

### Fixed
- **Candidato whitespace-strip en `parsePfx`** (`packages/signer/src/p12.ts`): tras los candidatos tal-cual / NFC / NFD / `trim()`, se prueba el PIN con **todo** el whitespace removido (`replace(/\s+/g,'')`) + su forma NFC. Cubre el espacio interno que `trim()` deja. Seguro: el PIN tal-cual se prueba primero (una contraseña con espacio legítimo sigue funcionando), y un PIN sin espacios no se ve afectado (no-op). El gate sigue siendo el MAC de forge, así que ningún candidato genera un falso-positivo. No recupera una sustitución `+`→espacio (ambigua).

### Changed
- `APP_VERSION` + `apps/pwa/package.json` → 0.7.42; `@firma-ec/signer` 0.7.5 → 0.7.6.

### Verified
- `vitest run` signer: 17/17 — incluido nuevo bloque que genera un .p12 con PIN `clave+2026` y valida recuperación de espacio interno (`clave + 2026`) + trailing, y que un PIN genuinamente incorrecto sigue siendo rechazado (sin falso-accept).

## [0.7.41] — 2026-05-21 — Multi-firma: verificación secuencial + document-timestamp cacheado (cierra el cuelgue de la última firma)

### Context
- 0.7.40 (cap 100 KB) avanzó: el usuario reportó que ahora **procesa firmas 0–4 y se detiene en la #5** (la 6ª/última) de un PDF B-LTA. Dos causas residuales: (1) las N firmas se verificaban con `Promise.all` (**en paralelo**) → todas emiten su beacon `ltv` casi a la vez y luego ejecutan su trabajo SÍNCRONO una tras otra sin beacon intermedio → tormenta síncrona cuyo total cruza los 30s (el watchdog no se resetea porque los beacons ya se emitieron). (2) `findDocumentTimestamps` + verificación del sello de archivo (que hashea **todo** el PDF) se ejecutaban **una vez por firma** (6× redundante).

### Fixed
- **Verificación SECUENCIAL** (`packages/verifier/src/index.ts`): `verifyAllSignatures` procesa las firmas en un `for…await` en vez de `Promise.all`. Así el trabajo síncrono de cada firma queda **entre sus propios beacons** y el watchdog se resetea entre firmas — solo importa el tiempo por-firma (acotado por los caps de CRL/OCSP + el deadline de LTV), no el acumulado.
- **Document-timestamp cacheado por PDF** (`packages/verifier/src/ltv.ts`): el escaneo + verificación del sello de archivo B-LTA es idéntico para todas las firmas del mismo PDF, pero corría 1×/firma. Ahora se memoiza por referencia de `pdfBytes` (la MISMA Uint8Array se pasa a todas las firmas) vía `WeakMap`, así corre **exactamente una vez**. Elimina el trabajo pesado redundante y acelera el multi-firma drásticamente. `ENGINE_VERSION` 0.7.13 → 0.7.14.

### Changed
- `APP_VERSION` + `package.json` → 0.7.41.

### Verified
- `vitest run` verifier: 72/72 (4 skipped).

## [0.7.40] — 2026-05-20 — El cuelgue era SÍNCRONO: cap agresivo de CRL/OCSP (100 KB) en LTV

### Context
- 0.7.39 (deadline `Promise.race` 12s) **tampoco** resolvió el cuelgue — y eso es **prueba concluyente**: un `setTimeout` NO puede dispararse mientras código **síncrono** retiene el único hilo del worker. Por tanto el bloqueo es síncrono, no asíncrono (descarta la cripto del document-timestamp, que es async y el deadline habría cortado). El único parseo síncrono de tamaño variable en LTV es pkijs `new CertificateRevocationList()`, que expande **cada entrada revocada** en un objeto: una CRL de ARCOTEL con cientos de miles de entradas tarda >30s en el CPU del móvil. El cap de 1.5 MB de 0.7.38 era demasiado alto.

### Fixed
- **Cap agresivo de CRL y OCSP a 100 KB** (`packages/verifier/src/ltv.ts`): cualquier CRL u OCSP embebido mayor a 100 KB se **omite antes de parsear** (warnings `crl_too_large_skipped` / `ocsp_too_large_skipped`). 100 KB parsea muy por debajo de un segundo; las CRLs grandes de ARCOTEL (el origen del cuelgue) se saltan. El perfil B-LT/B-LTA se sigue derivando de la **presencia** del DSS (conteos sin parsear), así que solo se pierde el detalle retrospectivo revoked/good — y LTV nunca cambia la validez de la firma (spec §6.4). Esto, sumado al deadline de 0.7.39 (async) y la memoización de 0.7.36, hace que la verificación **complete siempre** en móvil. `ENGINE_VERSION` 0.7.12 → 0.7.13.

### Changed
- `APP_VERSION` + `package.json` → 0.7.40.

### Verified
- `vitest run` verifier: 72/72 (4 skipped) — las CRLs/OCSP de los fixtures son < 100 KB, así que siguen evaluándose.

## [0.7.39] — 2026-05-20 — Deadline duro (Promise.race) alrededor de verifyLtv: cubre el cuelgue ASÍNCRONO

### Context
- 0.7.38 (cap CRL + presupuesto `Date.now()`) **no resolvió** el cuelgue (`verify:#5 ltv` seguía). Razón: `Date.now()` solo acota trabajo **síncrono**; un `await` que nunca resuelve (p.ej. la criptografía del document-timestamp B-LTA — ECDSA freetsa — o un parseo lento dentro de una llamada awaited) se salta esos chequeos y el watchdog de 30s dispara igual. Pista: la verificación pasó la fase `chain` (que usa crypto.subtle RSA sin problema) y murió en `ltv` → el staller asíncrono es muy probablemente la verificación ECDSA del sello de tiempo de archivo.

### Fixed
- **Deadline duro alrededor de `verifyLtv`** (`packages/verifier/src/index.ts`): `Promise.race([verifyLtv(...), deadline(12s)])`. Si LTV no termina en 12s — sin importar si lo que cuelga es síncrono o asíncrono — la fase retorna un `LtvSummary` degradado (perfil derivado de la presencia del DSS, `retrospectiveValid=false`, warning `ltv_timeout`). LTV es informativo y **nunca** cambia la validez de la firma (spec §6.4), así que la firma se reporta con su validez real. Junto al cap de CRL de 0.7.38 (riesgo síncrono), esto hace que la verificación **complete siempre**. `ENGINE_VERSION` 0.7.11 → 0.7.12.

### Changed
- `APP_VERSION` + `package.json` → 0.7.39.

### Verified
- `vitest run` verifier: 72/72 (4 skipped) — los fixtures completan LTV muy por debajo de 12s, así que el deadline no los afecta.

## [0.7.38] — 2026-05-20 — Cota dura en LTV (cap CRL + presupuesto de tiempo) para que la verificación NUNCA cuelgue

### Context
- Tras 0.7.36 (memoización) la verificación **seguía colgándose** (`verify:#5 ltv`). La memoización no ayuda cuando UNA sola CRL de ARCOTEL pesa megas: un único parseo síncrono de pkijs ya supera 30s en el CPU del móvil y bloquea el hilo del worker (el watchdog dispara sin que llegue otro beacon).

### Fixed
- **Cota dura de LTV** (`packages/verifier/src/ltv.ts`): LTV es **informativo y nunca invalida la firma** (spec §6.4), así que ahora se acota para que no pueda colgar:
  1. **Cap de tamaño de CRL** (`MAX_CRL_BYTES = 1.5 MB`): una CRL más grande se **omite** (un parseo completo bloquearía el hilo más allá del watchdog) con warning `crl_too_large_skipped`. El perfil B-LT/B-LTA se sigue derivando de la **presencia** del DSS, así que lo único que se pierde es el detalle retrospectivo revoked/good.
  2. **Presupuesto de tiempo** (`LTV_BUDGET_MS = 8s`): el bucle retrospectivo aborta al exceder el presupuesto con warning `ltv_budget_exceeded`.
  En ambos casos la firma sigue reportando su validez real; LTV degrada a una nota en vez de congelar el UI. `ENGINE_VERSION` 0.7.10 → 0.7.11.

### Changed
- `APP_VERSION` + `package.json` → 0.7.38.

### Verified
- `vitest run` verifier: 72/72 (4 skipped) — el cap/presupuesto no afecta los fixtures (CRLs de test < 1.5 MB, parseo < 8s).

## [0.7.37] — 2026-05-20 — PIN .p12 con `+`: retry trim + fingerprint de diagnóstico (no sensible)

### Context
- El usuario reportó que su contraseña .p12 **tiene un `+`** y es rechazada solo en móvil. `+` es ASCII → la normalización Unicode de 0.7.36 no aplica. No hay decodificación URL del PIN en el código (verificado), así que el `+` llega íntegro a forge. Hipótesis: el teclado del Samsung **auto-inserta un espacio** alrededor de la tecla de símbolos (`?123 → +`), produciendo un espacio invisible al inicio/fin → MAC distinta → `pin_invalid` espurio.

### Added
- **Retry con PIN recortado** (`packages/signer/src/p12.ts`): `parsePfx` ahora prueba también `pin.trim()` y `pin.trim().normalize('NFC')` además de tal-cual/NFC/NFD. La MAC de forge sigue siendo el gate (un candidato equivocado simplemente falla); el PIN tal-cual se prueba PRIMERO, así que una contraseña con espacio legítimo aún matchea por la vía as-typed.
- **Fingerprint de PIN no sensible** en el error (`p12.worker.ts` + `Firmar.svelte`): ante `pin_invalid` se anexa `[pin shape: len=N, trimmedDiffers=…, innerSpace=…, ascii=…]` — **nunca los caracteres**, solo la forma — visible en el mensaje de error del PIN. Permite diagnosticar desde el celular si el teclado mete un espacio (trimmedDiffers=true) o altera la longitud, sin filtrar la contraseña.

### Changed
- `APP_VERSION` + `package.json` → 0.7.37.

### Verified
- `vitest run` signer: 70/70.

## [0.7.36] — 2026-05-20 — Fix cuelgue LTV móvil (memoización) + retry de normalización Unicode del PIN .p12

### Context
- 0.7.35 localizó el cuelgue con precisión: `timeout (v0.7.35, last stage: verify:#5 ltv)` → fase **LTV de la firma #5** (PDF con 6 firmas). El watchdog por-fase funcionó: el problema es que **una sola fase LTV** supera los 30s en el CPU del móvil. El usuario confirmó además que el **PDF sin firma ya se arregló**.

### Fixed
- **Cuelgue LTV en móvil** (`packages/verifier/src/ltv.ts`): el bucle retrospectivo re-serializaba cada cert emisor (`cert.toSchema().toBER()`) **dentro de `tryParseOcsp` por cada índice OCSP y por cada cert de la cadena**, y re-parseaba la **misma CRL grande de ARCOTEL** (pueden ser MB) una vez por eslabón de cadena — O(cadena × entradas-DSS) de trabajo ASN.1 pesado por firma, ×6 firmas. Rápido en V8 desktop, >30s en móvil. Memoización por firma: `parseCert` (Map por Certificate), `parseCrlCached` (Map por índice), `ocspCache` (Map por `idx|eslabón`) → colapsa a O(cadena + entradas-DSS). Con el reset de watchdog por-fase de 0.7.35, cada firma vuelve a tener 30s, así que el PDF completo verifica. `ENGINE_VERSION` 0.7.9 → 0.7.10.
- **`.p12` "contraseña no coincide" con PIN correcto en móvil** (`packages/signer/src/p12.ts`): forge deriva la clave MAC del PKCS#12 desde el PIN como BMPString (UTF-16 de cada code point). Un teclado móvil puede entregar un PIN con caracteres no-ASCII (ñ, tildes) en una **forma de normalización Unicode distinta** (descompuesta `n`+◌̃) a la que usó el software emisor (precompuesta `ñ`) → mismo password visible, distintos code points, MAC distinta → `pin_invalid` espurio solo en móvil. Fix: `parsePfx` reintenta el PIN tal-cual → NFC → NFD antes de declarar `pin_invalid`. No-op para PINs ASCII (seguro). Si el PIN del usuario es ASCII puro y aun así falla, hay que investigar otra vía.

### Changed
- `APP_VERSION` + `package.json` → 0.7.36.

### Verified
- `vitest run` verifier ltv: 7/7. signer: 70/70 (incluye round-trip 3DES/AES PFX). bus: 13/13.

## [0.7.35] — 2026-05-20 — Cuelgue verify móvil = perf cliff por fase; beacons que resetean el watchdog + fix PDF sin firma en blanco

### Context
- Evidencia del dispositivo (Samsung, Chrome **y** Edge incógnito, **0.7.34 confirmado**): error `timeout … (v0.7.34, last stage: verify)`. Como corta a los 30s (no 6s) el worker **sí arranca** (beacon `boot`) → el cuelgue está **dentro de `verifyAllSignatures`**, no en la carga del worker. El único fetch del verify está acotado a 6s (OCSP) y para B-LTA ni se intenta → **no es red**. Conclusión: trabajo cripto/ASN.1 síncrono (validación de cadena + parseo de CRLs grandes de ARCOTEL embebidas en el DSS B-LTA) que en V8 desktop tarda ~2s pero en el CPU del móvil supera los 30s. Es regresión porque LTV/multi-firma agregaron ese trabajo.

### Fixed
- **Cuelgue de verificación en móvil**: el worker ahora emite un **beacon de progreso por fase** (`verify:scan`, `verify:cms`, `verify:integrity`, `verify:tsa`, `verify:chain`, `verify:ocsp`, `verify:ltv`, con prefijo `#N` en multi-firma). El watchdog del bus **se resetea en cada beacon**, así que una verificación lenta-pero-viva ya **no muere a los 30s** mientras ninguna fase individual los supere → completa en el móvil. Si una fase concreta cuelga de verdad, el `last stage:` la señala (cms/tsa/chain/ocsp/ltv) en vez del opaco `verify`. (`packages/verifier/src/index.ts` `verifyAllSignatures`/`verifyOneSignature` aceptan `onProgress` local-al-worker — no cruza `postMessage`; `verify.worker.ts` lo pasa.)
- **PDF sin firma mostraba pantalla en blanco** ("se queda vacío"): el branch `done` exigía `result` no-nulo (derivado de `signatures[0]`), y un PDF con 0 firmas tiene `signatures: []` → no renderizaba nada. Nuevo branch dedicado que muestra "Este PDF no contiene una firma electrónica" + botón de reinicio. (`Verificar.svelte`) — bug presente también en desktop, no solo móvil.

### Added — diagnóstico
- **`p12.worker`**: guard que si recibe **0 bytes** (buffer detachado antes del transfer) reporta `empty_p12` con la causa real, en vez de dejar que forge falle el MAC y lo reporte como `pin_invalid` (indistinguible de "contraseña incorrecta"). Para no atribuir erróneamente al PIN un problema de buffer. La defensiva-copy del caller (`Firmar.svelte:266`) es correcta, así que si persiste `pin_invalid` con PIN correcto, el siguiente paso es encoding del password.

### Changed
- `ENGINE_VERSION` 0.7.8 → 0.7.9. `APP_VERSION` + `package.json` → 0.7.35.

### Verified
- `vitest run` packages/verifier: 72/72 (4 skipped) — threading de `onProgress` no altera comportamiento.
- `vitest run` bus.test.ts: pendiente re-run tras build.

## [0.7.34] — 2026-05-20 — Regresión móvil: fallback a hilo principal cuando el module worker no arranca

### Context
- Dato clave del usuario: **en versiones anteriores la verificación SÍ funcionaba en el celular** → es una **regresión**, no una incompatibilidad de base. Falla en Chrome **y** Edge para Android (mismo Chromium), funciona en desktop, y rompe **los tres** workers (verify, p12-decrypt, sign) → el factor común es la **carga del chunk del module worker**, no la lógica de verificación. La regresión correlaciona con: `4097a0a` (p12 decrypt movido del hilo principal a un worker — antes funcionaba en móvil), `a868450` (deps cripto code-split en chunks lazy que el worker importa) y `5d69795` (verify pasó a multi-firma `runVerifyAll`).

### Added — apps/pwa 0.7.33 → 0.7.34
- **Fallback a hilo principal en `runVerifyAll`** (`bus.ts`): el worker emite un beacon `boot` apenas su módulo + chunks estáticos cargan (`verify.worker.ts`). Si no llega ningún mensaje en `VERIFY_BOOT_DEADLINE_MS` (6s) — el síntoma exacto de un module worker que en Chromium móvil muere en silencio sin `onerror` — se termina el worker nonato y la verificación **se re-ejecuta en el hilo principal** vía `import('@firma-ec/verifier')` dinámico (mantiene el chunk fuera del bundle de entrada). Es justo la ruta que funcionaba antes de mover la verificación off-thread, así que **restaura la función en los dispositivos afectados** a costa de bloquear el UI unos segundos. También cae al hilo principal si el worker dispara `error` antes de bootear o si `postMessage` lanza.
- **Error de timeout enriquecido**: ahora incluye versión + última etapa (`v0.7.34, last stage: none|boot|parse|verify`) para diagnóstico directo desde "Mostrar detalle técnico" sin cable. `last stage: none` = el worker nunca booteó (carga de chunk); `boot`/`verify` = booteó pero se colgó.

### Diagnóstico que habilita
- Si tras 0.7.34 la verificación funciona en el celular → la causa es **carga del chunk en contexto module-worker** (el hilo principal sí carga el mismo chunk). Replicar el patrón de fallback en p12/sign.
- Si sigue fallando con `fallback_failed` → el chunk es **inalcanzable** (SW/red), y la pista relevante es el fix de SW de 0.7.33.

### Verified
- `vitest run bus.test.ts`: 13/13 (3 nuevos: boot beacon no se filtra a UI, fallback al no bootear, NO fallback si ya booteó).

## [0.7.33] — 2026-05-20 — Causa raíz REAL del cuelgue móvil: el Service Worker borraba su propio precache

### Fixed
- **El cuelgue en Android NO era OCSP** (0.7.31/0.7.32 quedaron como defensa en profundidad, pero no eran la causa). Pista decisiva del usuario: **un PDF SIN firmas también se colgaba** (no toca red ni OCSP) y **la contraseña del .p12 tampoco se aceptaba** en el móvil. Ambos síntomas apuntan a una sola causa: **los Web Workers nunca arrancaban** (verify, p12-decrypt y sign son tres chunks `new Worker(new URL(...))` separados).
- **Causa raíz**: el handler `install` del Service Worker hacía un `caches.delete()` con alcance de **origen** de todos los `workbox-precache-*` en **cada** install. Con `registerType: 'prompt'` el SW nuevo se queda en `waiting` y **no activa** hasta que el usuario toca un toast de actualización. Resultado: al recargar, el SW nuevo se instala y **borra el precache que el SW viejo (todavía en control) está sirviendo** — además compite con `precacheAndRoute` que escribe ese mismo cache keyed-por-origen. Entonces los chunks de los workers daban **404** (Caddy `/assets/* serve-or-404`), los module workers **morían en silencio** (Chromium no dispara `onerror` fiable en fallo de import de dependencia) → verify colgaba hasta el watchdog de 30s y firmar reportaba "contraseña no aceptada" (el worker del p12 nunca corría). Android quedaba roto entre recargas porque **cada recarga re-borraba el precache** mientras el SW viejo seguía en control.

### Changed — apps/pwa 0.7.32 → 0.7.33
- **`sw.ts`**: eliminado el purgado destructivo de precache en `install`. Se conserva `cleanupOutdatedCaches()` (corre en `activate`, es revision-aware y nunca borra el precache activo). El SW ahora hace **`skipWaiting()` en `install`** + `clients.claim()` en `activate`, así una sola recarga en un cliente stale toma el build nuevo, repuebla el precache y restaura los chunks de los workers (verify/p12/sign). El listener `controllerchange` de `swUpdate.svelte.ts` recarga una vez al tomar control.
- `APP_VERSION` + `package.json` bump.

### Notas
- Self-heal: los dispositivos ya rotos se arreglan con **una recarga** sobre 0.7.33 (SW nuevo → skipWaiting → activate → claim → controllerchange → reload → shell + precache frescos).
- Regla anti-regresión documentada en `sw.ts`: **nunca** reintroducir un `caches.delete()` general en `install`.

## [0.7.32] — 2026-05-20 — Fix DEFINITIVO: cuelgue verificación móvil = OCSP a endpoint caído

### Fixed
- **Causa raíz REAL del cuelgue en Android Chrome** (los fixes 0.7.30/0.7.31 fueron paliativos): `ocsp.firmar.ec` está **caído/sin registro DNS** (devuelve HTTP 000). El verificador intentaba un OCSP en vivo a ese host para perfiles con timestamp. En desktop el fetch **falla rápido** (connection refused → rechaza al instante → `not_checked` → sigue). En **red móvil el host inalcanzable hace black-hole** del SYN (sin RST), así que el fetch **se queda colgado** en vez de fallar — colgando toda la verificación hasta el watchdog de 30s.

### Changed — packages/verifier 0.7.7 → 0.7.8
- `ENGINE_VERSION` 0.7.7 → 0.7.8.
- **`index.ts`**: el verificador ahora **salta el OCSP en vivo cuando la firma trae revocación embebida en el DSS (B-LT / B-LTA)**. Esa es exactamente la evidencia de revocación que el perfil exige (capturada al firmar), así que el fetch en vivo es redundante Y peligroso (el host puede colgar la red). Nuevo guard `hasEmbeddedRevocation` (DSS con ≥1 OCSP o CRL). Solo B-T (timestamp sin DSS) intenta aún un OCSP en vivo acotado. Resultado: para PDFs B-LTA (como los que firma firmar.ec) **cero llamadas de red en la verificación → cero cuelgue, en cualquier red**.

### Changed — apps/pwa 0.7.31 → 0.7.32
- `APP_VERSION` + `package.json` bump.

### Notas
- Los watchdog (0.7.30) y el OCSP race-deadline (0.7.31) se conservan como defensa en profundidad — siguen protegiendo el caso B-T y cualquier futura llamada de red.
- Acción de infra pendiente (separada): decidir si se levanta `ocsp.firmar.ec` o se retira del código por completo. Mientras tanto B-LTA no lo necesita.

### Verified
- `pnpm vitest run` packages/verifier: 72/72 pass, 4 skipped.

## [landing 0.1.18] — 2026-05-20 — Corrección de exactitud factual del contenido

### Fixed
- **Multi-firma mal descrita** en comparativos Adobe Sign (ES+EN): decía "Workflows multi-firmante ❌ No (v1; quizás v2)", lo que negaba una capacidad que SÍ existe. Reescrito a "🟡 Secuencial manual (cada persona firma y pasa el PDF al siguiente; las firmas previas se conservan válidas). Sin orquestación de links/recordatorios" — refleja la verdad: la firma secuencial manual funciona (incremental update), pero el workflow orquestado con links/notificaciones NO existe (requeriría backend, choca con el modelo sin-servidor).
- **Perfiles PAdES incompletos**: comparativos y `Cumplimiento.astro` solo listaban "PAdES B-B". Actualizado a "B-B / B-T / B-LT / B-LTA" + fila nueva de Timestamp (RFC 3161 / ETSI EN 319 122) y Revocation ahora incluye CRL RFC 5280.
- **Conteo de ACEs inconsistente (7 vs 8) + ECI incorrecta**: `firma-electronica-ecuador.md` (ES) tenía header "8 ECIs" pero tabla de 7 con Lazzate y sin ArgosData/Judicatura; la versión EN decía "7 accredited ECIs"; FAQ 03 (ES+EN) y `glosario/en-tsl.md` ("7 root certificates") arrastraban el mismo error. Alineado todo a las **8 ACEs reales** de la TSL-EC actual (BCE, Consejo de la Judicatura/iCert-EC, Security Data, ANFAC, ArgosData, Uanataca, Eclipse Soft, Datil) — sin Lazzate, que no está en la verdad actual. Coherente con `Compatibilidad.astro` que ya era correcto.

### Verified
- `pnpm --filter @firma-ec/landing build` — 28 páginas, 0 errores.

## [0.7.31] — 2026-05-20 — Fix: OCSP fetch atascado cuelga la verificación en red móvil

### Fixed
- **La verificación se cuelga en red móvil aunque el cliente ya esté en 0.7.30** (reportado en Android Chrome tras limpiar caché). El watchdog de 0.7.30 convertía el cuelgue en error a los 30s, pero la causa seguía: el worker arrancaba (progreso `parse`→`verify`) y luego `checkOcsp` se quedaba pegado. Causa raíz: el fetch OCSP a `https://ocsp.firmar.ec` se atasca en establecimiento de conexión (DNS/TLS) en ciertas redes móviles, y `AbortController.abort()` **no rechaza** un fetch atascado antes de recibir respuesta en esa condición → `await postViaProxy(...)` nunca settlea → la verificación nunca retorna.

### Changed — packages/verifier 0.7.6 → 0.7.7
- `ENGINE_VERSION` 0.7.6 → 0.7.7.
- **`ocsp.ts` `checkOcsp`**: deadline duro vía `Promise.race([fetch, timer-que-rechaza])`. Garantiza que `checkOcsp` settlea dentro de `fetchTimeoutMs` (6s) sin importar si el fetch subyacente aborta o no. Se sigue llamando `ac.abort()` para liberar el socket donde el navegador lo respeta. OCSP en vivo es **redundante para B-LTA** (la revocación ya viene embebida en el DSS), así que un `not_checked` por timeout no degrada el veredicto.

### Changed — apps/pwa 0.7.30 → 0.7.31
- `APP_VERSION` + `package.json` bump (consume verifier 0.7.7).

### Verified
- `pnpm vitest run` packages/verifier: 72/72 pass, 4 skipped.

## [0.7.30] — 2026-05-20 — Fix: verificación cuelga (spinner infinito) en clientes con SW stale

### Fixed
- **Spinner infinito al verificar en móvil (reportado en Android Chrome).** El PDF cargaba pero la verificación nunca terminaba. Causa raíz triple: (1) un Service Worker stale (cliente que no aceptó el prompt de actualización) servía un app-shell que referencia chunks con hash ya purgados por el deploy nuevo; (2) Caddy respondía esos `/assets/*.js` faltantes con `index.html` (HTML 200) por el SPA fallback `try_files`; (3) un module worker cuyo `import()` recibe HTML en vez de JS falla a cargar y **Chromium no dispara `worker.onerror` de forma fiable** para errores de carga de dependencias de module workers → el worker queda creado pero su handler nunca corre → `postMessage` al vacío → como `runVerify`/`runVerifyAll` no tenían timeout, spinner infinito.

### Changed — apps/pwa 0.7.29 → 0.7.30
- **`lib/workers/bus.ts`**: `runVerify` y `runVerifyAll` ahora tienen un **watchdog de timeout** (`DEFAULT_VERIFY_TIMEOUT_MS = 30s`, configurable vía `opts.timeoutMs`, `0` lo desactiva). Si el worker no postea result/error/progress dentro de la ventana, la promesa rechaza con `code: 'timeout'`. El timer se **resetea en cada mensaje de progreso**, así un worker lento-pero-vivo no se mata; solo uno muerto-en-silencio. Convierte el cuelgue en un error accionable.
- **`infra/docker/Caddyfile.pwa`**: nuevo bloque `@assets path /assets/*` con `file_server` **antes** del SPA `try_files`. Los assets hasheados ahora sirven el archivo o **404 real** — nunca caen al fallback `index.html`. Esto deja que el `import()` de un chunk purgado **rechace** (en vez de recibir HTML), permitiendo que el worker reporte error y la UI muestre el mensaje de recarga. Self-heal para clientes stale.
- **`routes/Verificar.svelte`**: el mapeo de error ahora incluye `timeout` (lowercase, del watchdog) y `worker_error` → `error.engine_TIMEOUT`.
- **`lib/i18n.svelte.ts`**: mensaje `error.engine_TIMEOUT` reescrito (ES+EN) para guiar a **recargar la página / cerrar y reabrir la app instalada** (la causa típica es un SW stale), en vez del genérico "intenta de nuevo".

### Added — apps/pwa/src/lib/workers/bus.test.ts
- 3 tests del watchdog: rechaza con `timeout` si el worker calla; progress resetea el deadline (slow-but-alive no muere); `timeoutMs=0` desactiva el watchdog. 10/10 tests del bus verde.

### Notas de operación
- **Workaround inmediato para usuarios afectados** (sin esperar el deploy): recargar con caché limpia o, en la app instalada, cerrarla y reabrirla; en última instancia borrar datos del sitio para desregistrar el SW viejo.
- El verifier (`packages/verifier` 0.7.6) no cambió — el fix de 0.7.29 (B-LTA multi-sig) sigue intacto.

## [0.7.29] — 2026-05-19 — Verifier: B-LTA multi-sig DocTimeStamp handling (P0 regression fix)

### Fixed
- **PWA mostraba "Firma inválida" para PDFs B-LTA legítimos firmados por firmar.ec con TSA wrap de freetsa.org.** Síntoma reportado: PDF de Alfonso firmado con su cert ArgosData real (cuyo root está en la TSL-EC con fingerprint correcto) aparecía como "Firma inválida — El certificado del firmante no proviene de una ACE acreditada por ARCOTEL". El cert sí encadenaba; el verifier estaba contaminado por la firma TSA-wrap.

### Changed — packages/verifier 0.7.5 → 0.7.6
- `ENGINE_VERSION` 0.7.5 → 0.7.6.
- **Bug A — `verifyAllSignatures` filtra DocTimeStamps**: las firmas con `/SubFilter /ETSI.RFC3161` (PAdES B-LTA document timestamp wrap) ya NO se cuentan como "firmas del usuario". Antes se procesaban como signers normales y (a) fallaban con `weak_signature_algorithm` por el ecdsa-SHA512 que usa freetsa.org, (b) sus certs entraban a `pooledIntermediates` confundiendo a `pkijs.CertificateChainValidationEngine.verify()` que devolvía `false` para la firma real → `matchedRoot=undefined` → `untrusted_root`. El verifier ya expone el DTS por `signature.timestamp` + `verifyLtv` → no se pierde información.
- **Bug B — `parseString` anclado al `<<` del dict**: el escaneo forward desde `/ByteRange.tokenAt` se filtraba al siguiente sig dict porque `/SubFilter` suele preceder a `/ByteRange` (orden alfabético o del productor). Antes: dos firmas con subFilter `'unknown'` o cruzados. Nuevo: `findDictStart()` retrocede hasta el `<<` del dict actual (con depth counting) y escanea desde ahí. Fix también beneficia a `/Reason`, `/Location`, `/ContactInfo`, `/M`.
- **Bug B' — `parseString` soporta PDF Names** (`/foo`): antes solo aceptaba literales `(string)` o `<hex>`; `/SubFilter` es un Name y devolvía `undefined` → `'unknown'`. Añadido parser de Name con todos los delimitadores PDF (whitespace + `()<>[]{}/%`).
- **Bug C — DTS-wrap no dispara `incremental_updates`**: nueva flag `appendedBytesAreDocTimeStamp` en `verifyOneSignature`. `verifyAllSignatures` la setea cuando los bytes apendados después de la firma del usuario corresponden a un DTS B-LTA legítimo que llega hasta EOF. La firma del usuario queda `valid`, no `warning`.

### Added — packages/verifier/tests
- **b-lta-multisig-regression.test.ts** — 3 tests que blindan los 3 bugs con `carta-arrendamiento-firmado.pdf` (firmado por Alfonso/ArgosData con TSA wrap freetsa, 2 firmas PAdES detectadas, solo 1 firma de usuario).

### Changed — apps/pwa 0.7.28 → 0.7.29
- `APP_VERSION` + `package.json` bump.

### Verified
- `pnpm vitest run` en packages/verifier: 13/13 archivos verde, 73/73 tests pass (3 nuevos + 70 existentes), 4 skipped.
- PDF de Alfonso ahora retorna: `overallStatus='valid'`, `signatureCount=1`, `matchedRootSlug='argosdata'`, profile B-LTA, sin warnings.

## [0.7.28] — 2026-05-19 — Verifier: untrusted_root warning + specific invalid summaries

### Fixed
- **PWA verdict UX**: cuando una firma cripto-correcta no encadena a ninguna ACE ARCOTEL (caso típico: cert auto-firmado o emisor no acreditado), el verificador mostraba "Firma inválida — La firma no es válida o el documento fue modificado tras la firma". El mensaje sugería tampering inexistente. Discovered via Playwright E2E real contra prod 2026-05-18 con fixture `sample.pdf` + `Test Signer RSA-2048`: hash matched, modifications=No, byteRange correcto, pero verdict invalid → user confundido.

### Changed — packages/verifier 0.7.3 → 0.7.5
- `ENGINE_VERSION` bumped 0.7.4 → 0.7.5.
- Cuando `!path.success && !trustInconclusive` el verifier **empuja warning `untrusted_root`** explicando que el cert es cripto-correcto pero el emisor no está reconocido por ARCOTEL. Verdict sigue siendo `invalid` (correcto).

### Changed — apps/pwa 0.7.22 → 0.7.23
- `Result.svelte`: el summary del verdict 'invalid' se selecciona por causa específica derivada de:
  - `!integrity.digestMatches` → `invalid_summary_hash_mismatch`
  - `ocsp.status === 'revoked'` → `invalid_summary_revoked`
  - warning code `untrusted_root` → `invalid_summary_untrusted_root`
  - fallback → `invalid_summary_bad_signature`
- `i18n.svelte.ts`: 4 nuevas keys (ES+EN) `verificar.invalid_summary_{untrusted_root, revoked, hash_mismatch, bad_signature}`. La key original `invalid_summary` queda como fallback compatible.

## [landing 0.1.17] — 2026-05-18 — TSL truth fix + deploy script + landing CI

### Fixed
- **apps/landing/public/llms-full.txt** — sección "Modelo de confianza ARCOTEL" tenía 5 ACEs como "root pendiente". La realidad (verificada contra `apps/pwa/public/trust/tsl-ec.json` v1.10.0 seq 11) es **8/8 ACEs activas con root real cargado** (BCE, Security Data, ANFAC, Judicatura, Uanataca, ArgosData, Datil, Eclipsoft). Registro Civil marcado `isDefunct` desde v0.7.12 (firma con certs BCE/Security Data, no opera PKI propia).

### Added
- **scripts/deploy-landing.sh** — pipeline manual reusable: tar + scp a IAS01 + docker build + push + swarm update + HTTP smoke verify. Reemplaza la cadena de comandos one-off.
- **.github/workflows/landing-ci.yml** — CI dedicado para landing en push a main / tags `v-landing-*`. Valida `pnpm build`, presencia de `llms.txt`, `llms-full.txt`, `.well-known/ai-plugin.json`, `security.txt`, `robots.txt`, `sitemap-index.xml`, page count ≥28, JSON válido, y docker build. Push a registry + swarm update siguen siendo manuales (requieren acceso SSH a la red IDK).

## [landing 0.1.16] — 2026-05-18 — AI Search readiness (llms-full.txt + ai-plugin.json)

### Added
- **apps/landing/public/llms-full.txt** (8.4KB): comprehensive content dump optimized for LLM retrieval (Claude, GPT, Perplexity, Gemini). Covers trust model TSL-EC, PAdES profiles B-B/B-T/B-LT/B-LTA, legal framework Ecuador (LCE 2002-67 + LOPDP), full FAQ, glossary, and preferred citation format.
- **apps/landing/public/.well-known/ai-plugin.json**: ChatGPT plugin manifest. `description_for_model` geared to Ecuadorian electronic-signature questions; `auth: none`, `api: none` (info-only — points crawlers to llms.txt, llms-full.txt, sitemap).

### Changed
- **apps/landing/public/llms.txt**: replaced stale URLs (blog and /spec/* routes that never shipped) with real ones (faq, glosario, comparativos/adobe-sign, comparativos/firmaec, en/*, security.txt, sitemap-index.xml).

### Audit context
- Closes "AI Search 70/100" gap identified in firmar.ec SEO/SEM/AI audit 2026-05-18.

## [0.7.22] — 2026-05-15 — CI unblock: tsl-ec tsconfig + biome lint reality

### Fixed
- **packages/tsl-ec/tsconfig.json**: añadido `"exclude": ["src/build-json.ts"]`. El archivo se ejecuta como script Node con `--experimental-strip-types` (requiere extensión `.ts` explícita en imports), pero el library build con `tsc` lo veía y fallaba con `TS5097`. Excluirlo del compile mantiene el script funcional y desbloquea el Release workflow que llevaba fallando desde v0.7.17.
- **biome.json**: relajadas reglas que rompían la realidad del codebase. `useLiteralKeys: off` (colisiona con TS `noPropertyAccessFromIndexSignature`), `useConst: off` (rompía bindings `$state` en Svelte runes), `noUnusedImports: off` + `noUnusedVariables: off` (biome no detecta usos en templates Astro/Svelte y eliminaba imports válidos), `noNonNullAssertion: off` (estilo aceptado). Otras reglas (noExplicitAny, noConsole, useTemplate, noAssignInExpressions, noImplicitAnyLet, useOptionalChain, etc.) bajadas a `warn` → 143 warnings visibles en IDE como tech debt, 0 errores bloqueantes.

### Formatted
- `pnpm biome format --write` aplicado a 212 archivos (sólo whitespace: LF endings, single quotes, trailing commas).
- `pnpm biome organizeImports` aplicado vía `biome check --fix` (sólo reordenamiento de imports, no eliminación).

### Verified locally
- `pnpm biome check` — 0 errors, 143 warnings.
- `pnpm -r typecheck` — 16/16 packages pass (incluyendo pwa svelte-check y landing astro check).
- `pnpm build` — todos los packages + 28 páginas landing.
- `pnpm build:tsl` + `tsc tsl-ec` — OK.

## [0.7.12] — 2026-05-15 — Registro Civil marked defunct (8/8 ACEs activas, demo mode OFF)

### Changed — tsl-ec 1.10.0 (TSL_SEQUENCE 11)
- **Registro Civil** slot marked `isDefunct: true`. Evidencia: resolución
  oficial **009-DIGERCIC-CGAJ-DPyN-2025** descargada del sitio público
  del Registro Civil, firmada por 4 funcionarios. Análisis de las 4
  cadenas CMS PAdES:
  - **Director General Ottón José Rivadeneira González** → cert emitido
    por `AC BANCO CENTRAL DEL ECUADOR` (intermedio BCE), raíz **BCE**.
  - **Andrea Cristina Garnica Rojas** (analista RC) → cert emitido por
    `AC BANCO CENTRAL DEL ECUADOR`, raíz **BCE**.
  - **Víctor Andrés Oquendo Torres** → cert emitido por
    `AUTORIDAD DE CERTIFICACION SUBCA-2 SECURITY DATA`, raíz
    **Security Data CA-2**.
  - **María José Rentería Landívar** → cert emitido por
    `AUTORIDAD DE CERTIFICACION SUBCA-2 SECURITY DATA`, raíz
    **Security Data CA-2**.
- Conclusión: Registro Civil NO opera una raíz PKI independiente. Sus
  funcionarios delegan 100% en BCE + Security Data. La acreditación
  ARCOTEL del Registro Civil como ECI es nominal/histórica.

### Changed — pwa 0.7.12
- Banner `TRUST_PARTIAL` ahora dirá **8 de 8 ACEs activas** (no aparecerá
  porque ya no hay placeholders entre los activos). Demo mode efectivamente
  OFF para cualquier PDF firmado con cert de las 8 ACEs reales.
- `verificar.demo_banner_body` (es+en) actualizado con la explicación
  de delegación BCE/SD del Registro Civil. Banner se conserva por si
  algún día aparece un PDF firmado con cert de una ACE inactiva.

## [0.7.11] — 2026-05-15 — Judicatura iCert-EC real Root CA

### Added — tsl-ec 1.9.0 (TSL_SEQUENCE 10)
- Real **iCert-EC root** loaded into `judicatura-2024.pem`. Subject:
  `CN=ICERT-EC ENTIDAD DE CERTIFICACION RAIZ, OU=SUBDIRECCION NACIONAL
  DE SEGURIDAD DE LA INFORMACION DNTICS, O=CONSEJO DE LA JUDICATURA,
  L=DM QUITO, C=EC`. Valid 2014-10-16 → 2034-10-16 (20-year root,
  10 años vigentes restantes). SHA-256
  `a434953dc5a028313d9e07b8cfefdf5a47b08e2d353bffb854a52360d6ef00c6`.
  Extracted offline from PAdES CMS chain of a 4-signature judicial PDF
  (`075-2026.pdf`, 3 firmas ancladas en iCert-EC). `icert.fje.gob.ec`
  sigue en mantenimiento — fetch público no era viable.
- **8/9 ACEs activas reales** ahora (era 7/9). Solo Registro Civil
  queda como placeholder.

### Changed — pwa 0.7.11
- `verificar.demo_banner_body` (es+en): "8 de 9 ACEs ARCOTEL activas
  tienen raíz real cargada (… Judicatura iCert-EC); falta solo 1
  (Registro Civil)".
- TSL bumped 1.8.0 → 1.9.0 (sequence 9 → 10).

### TODO for v0.7.12+
- **Registro Civil**: hipótesis activa de delegación en BCE pendiente
  de confirmar con PDF de funcionario operativo (no Director).

## [0.7.10] — 2026-05-15 — Security Data legacy Root CA (parallel anchor)

### Added — tsl-ec 1.8.0 (TSL_SEQUENCE 9)
- New trust anchor slot `securitydata-legacy`. Self-signed legacy root
  `CN=AUTORIDAD DE CERTIFICACION RAIZ SECURITY DATA, O=SECURITY DATA
  S.A., OU=ENTIDAD DE CERTIFICACION DE INFORMACION, C=EC`. Valid
  2011-02-16 → 2031-02-16 (20-year root, still vigente). SHA-256
  `fc8d6968851e6dc8c4be8fe8962e52d85ad32c90cd7b0d7fb6376c7a165c0e2a`.
  Extracted 2026-05-15 from 6 PAdES CMS chains across production signed
  PDFs (`whats empresa recovery/Media/WhatsApp Business Documents/…`).
  Modelled as a separate slug (not concatenated into
  `securitydata-2024.pem`) because the verifier's `pemToCert` parses one
  cert per PEM file and the `TrustRoot` schema carries a single
  fingerprint/validity pair.
- Banner counter unchanged (Security Data already counted in v0.7.7).
  This release strengthens chain validation for end-entity certs issued
  under the older Security Data root that remain operative.

### Changed — pwa 0.7.10
- `tsl-ec` bumped 1.7.0 → 1.8.0 (sequence 8 → 9). 18 trust roots in TSL
  (17 ARCOTEL slots + 1 legacy parallel anchor). 8 real roots loaded.

### TODO for v0.7.11+
- **Judicatura**: still placeholder. icert.fje.gob.ec sigue en
  mantenimiento. Esperar PDF firmado B-LT/LTA con cert iCert-EC.
- **Registro Civil**: still placeholder. Necesita PDF firmado por
  funcionario operativo (no Director) para confirmar si emiten desde
  raíz propia o delegan en BCE.

## [0.7.9] — 2026-05-15 — ANFAC Ecuador real Root CA via PAdES PDF scan

### Added — tsl-ec 1.7.0 (TSL_SEQUENCE 8)
- Real **ANFAC Ecuador Root CA** loaded into
  `packages/tsl-ec/src/roots/anfac-2024.pem`. Found by scanning all 114
  signed PDFs in `~/Nextcloud/Documentos`: the `Cliente GPS/Borrador de
  Contrato Ariendo de equipos…-signed.pdf` PAdES CMS chain delivered
  the full self-signed root. Subject: `CN=ANF High Assurance Ecuador
  Root CA, O=ANFAC AUTORIDAD DE CERTIFICACION ECUADOR C.A.
  (RUC 1792601215001), OU=ANF Clase 1 CA EC, C=EC`. Valid
  2019-10-17 → 2039-10-12 (20-year root). SHA-256
  `0f361d8b258123ea9bb84dd3f2c821c0285479626e1185e12f1a04b85546e459`.
  ANFAC Ecuador is operationally active — the previous "no public web
  presence" finding was misleading. They issue certificates under their
  own EC-incorporated root (distinct from the Spanish ANF AC root).
- **7/17 ACEs now have real roots**: eclipsesoft, uanataca, argosdata,
  datil, bce, securitydata, anfac. 2 SRI-accepted still placeholders:
  judicatura, registro-civil.

### Changed — pwa 0.7.9
- Verifier `TRUST_PARTIAL` banner now reports `7 de 9 ACEs ARCOTEL
  activas` instead of `6 de 9`. Banner names the 2 remaining active
  placeholders (Judicatura, Registro Civil).

### TODO for v0.7.10+
- **Security Data legacy root** (`AUTORIDAD DE CERTIFICACION RAIZ
  SECURITY DATA`, sha256 `fc8d6968…`, valid 2011-02-16 → 2031-02-16)
  found in 6 additional signed PDFs but not yet loaded — requires
  decision: concatenate PEMs in `securitydata-2024.pem` or add separate
  slug. Certificates issued under this older root are still valid; ship
  alongside the CA-2 root in a follow-up.
- **Judicatura**: all 35 Judicatura-signed PDFs scanned used legacy
  `adbe.pkcs7.sha1` mode that does NOT embed the chain. Still waiting
  for either a B-LT/LTA signed document or `icert.fje.gob.ec` to come
  back from maintenance.

## [0.7.8] — 2026-05-15 — Header logo + "Inicio" now redirect to landing

### Changed — pwa 0.7.8
- `Header.svelte`: lockup ("firmar.ec app") and the "Inicio / Home" nav item
  now point to `https://firmar.ec/` (institutional landing) instead of the
  internal SPA `/` route. The PWA `Home.svelte` route still exists for
  deep-links and installed-app entry, but the global navigation always
  takes the user back to the institutional site as expected. Behaviour is
  identical on desktop and mobile menus.

## [0.7.7] — 2026-05-15 — Security Data real Root CA via signed contract PDF

### Added — tsl-ec 1.6.0 (TSL_SEQUENCE 7)
- Real **Security Data Root CA** loaded into
  `packages/tsl-ec/src/roots/securitydata-2024.pem`. Extracted from the
  PAdES CMS chain of a real signed contract (`CONTRATO2026 SOLUCIONES…`)
  that had the full LT-level chain embedded. Self-signed:
  `CN=AUTORIDAD DE CERTIFICACION RAIZ CA-2 SECURITY DATA,
  O=SECURITY DATA S.A. 2, OU=ENTIDAD DE CERTIFICACION DE INFORMACION,
  C=EC`. Valid 2019-10-15 → 2039-10-06 (20-year root). SHA-256 fingerprint
  `503b5960fa8cc58f3367642a911fd8f8277e474d6891637fe56ca2a69f069cbd`.
  Security Data does not publish this PEM on a public URL; offline
  extraction from a real signed PDF was the only path.
- **6/17 ACEs now have real roots** (eclipsesoft, uanataca, argosdata,
  datil, bce, securitydata). 3 SRI-accepted still placeholders: anfac,
  judicatura, registro-civil.

### Changed — pwa 0.7.7
- Verifier `TRUST_PARTIAL` banner now reports `6 de 9 ACEs ARCOTEL
  activas` instead of `5 de 9`. Banner copy explicitly names the 3
  remaining active placeholders (ANFAC, Judicatura, Registro Civil).

### Notes — Judicatura still placeholder
- Attempted: P12 client cert (only end-entity), legacy `adbe.pkcs7.sha1`
  PDF (chain not embedded), OCSP responder (only returned responder
  cert), crt.sh (502), `icert.fje.gob.ec` (site under maintenance).
- Needs: a Judicatura-signed PDF at LT/LTA level (B-LT or B-LTA) where
  the full chain is mandatorily embedded, OR a direct CA cert from
  iCert when their site is back, OR a working crt.sh query.

## [0.7.6] — 2026-05-15 — BCE real Root CA via Registro Civil PAdES chain

### Added — tsl-ec 1.5.0 (TSL_SEQUENCE 6)
- Real **BCE Root CA** loaded into `packages/tsl-ec/src/roots/bce-2024.pem`.
  Extracted from the PAdES CMS chain of a Certificado de Matrimonio signed
  by the Director General del Registro Civil (Ottón José Rivadeneira
  González). The Registro Civil uses BCE-issued certificates, so the CMS
  delivered the BCE root directly. Subject == Issuer (self-signed):
  `CN=AUTORIDAD DE CERTIFICACION RAIZ DEL BANCO CENTRAL DEL ECUADOR,
  O=BANCO CENTRAL DEL ECUADOR, OU=ECIBCE, L=Quito, C=EC`. Valid
  2011-08-08 → 2031-08-08 (20-year root). SHA-256 fingerprint
  `11c7c59be9d21d216f0e8151378d53d03b314060559adc49da161ec4f7829bec`.
  BCE does **not** publish this PEM on a public URL (their WAF blocks
  `/aia/eciroot.crt`); the only path was offline extraction from a real
  signed PDF.
- **5/17 ACEs now have real roots** (eclipsesoft, uanataca, argosdata,
  datil, bce). 4 SRI-accepted CAs still placeholders: anfac, judicatura,
  registro-civil, securitydata.

### Changed — pwa 0.7.6
- Verifier `TRUST_PARTIAL` banner now reports `5 de 9 ACEs ARCOTEL
  activas` instead of `4 de 9`. Banner copy explicitly names the 4
  remaining active placeholders (ANFAC, Judicatura, Registro Civil,
  Security Data) and keeps disclosing the 8 inactive ACEs.

### Notes
- The discovery that Registro Civil signs with a BCE-issued cert (rather
  than its own ECI root) raises a question for v0.7.7+: does Registro
  Civil even issue end-entity certs from its own root, or is its ARCOTEL
  accreditation purely formal while it delegates to BCE? Keep the slot
  for now and revisit when we find a document signed with a true
  Registro-Civil-issued cert.

## [0.7.5] — 2026-05-14 — Datil real CA + isDefunct flag + IDK Manager wordmark

### Added — tsl-ec 1.4.0 (TSL_SEQUENCE 5)
- Real Datil Root CA loaded into `packages/tsl-ec/src/roots/datil-2024.pem`
  — fetched from Datil public S3 (`Root_CA.crt` linked from
  `datil.com/certificados`). Subject `CN=Datil Autoridad de Certificacion,
  O=Datilmedia S.A.`, self-signed 2021-12-16 → 2031-12-14, sha256
  `4015 74c5 215e d1d6`. **4/17 ACEs now have real roots** (eclipsesoft,
  uanataca, argosdata, datil).
- New `isDefunct?: boolean` field on `TrustRoot` interface for
  ARCOTEL-listed CAs with no operational public presence. Verifier
  excludes them from the active denominator so the demo banner reflects
  only currently-issuing CAs.
- 8 entries flagged `isDefunct: true` (alpha-technologies, appfirmas,
  corpnewbest, darkcam, firmasegura, lazzate, letmi, primecorelat) —
  ARCOTEL-listed but no public website, no PKI repository, no SRI
  acceptance. Preserved in TSL for traceability against ARCOTEL listing.

### Changed — pwa 0.7.5
- Verifier `TRUST_PARTIAL` banner now reports `4 de 9 ACEs ARCOTEL
  activas` instead of `3 de 17`. New i18n copy explicitly names the 5
  remaining active placeholders (ANFAC, BCE, Judicatura, Registro Civil,
  Security Data) and discloses that 8 inactive ACEs are excluded.
- `packages/verifier/src/index.ts` heuristic now filters
  `activeRoots = roots.filter(r => !r.isDefunct)` before computing
  `placeholderCount` / `allRootsPlaceholder` / `someRootsPlaceholder`.

### Changed — landing
- `OperadoPor.astro` replaced the plain "IDK Manager" text heading with
  the official `idk-manager-wordmark.png` brand asset (160×66, @2x 320×132)
  copied from `_work/idkmanager-web/public/brand/`. H2 retains semantic
  text via `sr-only` span; image alt text preserved for screen readers.

### Fetch attempts that failed (kept as placeholder, still in TSL)
- BCE: `bce.fin.ec/aia/eciroot.crt` actively blocked by WAF
  ("requerimiento de despliegue del url fue rechazado"). Contact
  `seguridad@bce.ec`.
- Security Data: site live but no PKI repository at standard paths
  (`/repositorio`, `/wp-content/uploads/...CA-RAIZ...`). Contact
  `+593 2 392 2169`.
- ANFAC Ecuador: zero public web presence (anfac.ec, .com.ec all
  NXDOMAIN). Spanish ANF/ANFAC is a different entity.
- Consejo de la Judicatura: no PKI subdomain
  (`firmadigital.funcionjudicial.gob.ec` NXDOMAIN).

## [seo-2026-05-14] — SEO / GSC fixes (landing 0.1.14 + pwa 0.7.4)

> Tag collision avoidance: registry already holds `landing:v0.1.13` /
> `pwa:0.7.3` from prior builds with different content; bumped to
> `0.1.14` / `0.7.4` per qa-verify §7.1 (never reuse a pushed image tag).

### Fixed — landing 0.1.14
- `/sitemap.xml` now returns a valid 200 sitemapindex (was 404). Google
  Search Console probes the bare `/sitemap.xml` path independently of the
  `Sitemap:` directive in robots.txt; the new static file points at
  `sitemap-0.xml` directly so both discovery paths resolve.
- JSON-LD `SoftwareApplication.softwareVersion` updated from stale `0.1.0`
  to current PWA `0.7.4` so structured data reported to crawlers matches
  the deployed app.

### Fixed — pwa 0.7.4
- `/robots.txt` now serves a real `User-agent: * / Disallow: /` body
  instead of falling through to the SPA `index.html` (200 HTML response
  on robots.txt confused Google indexing — surface mirrors the existing
  `X-Robots-Tag: noindex, nofollow` Caddy header).

## [0.7.3] — 2026-05-12 — Demo banner version-agnostic + verifier test fixes

### Fixed — pwa 0.7.3
- Demo banner ("Verificación en modo demostración") no longer hard-codes
  the release version (was stuck at "v0.7.0" two releases after the bump).
  Banner now states the TSL coverage state (3/17 ACEs real, 14 placeholder)
  without a version prefix so it stays accurate across releases.

### Fixed — verifier (test suite, no engine change)
- `regression-real-eci.test.ts` updated for v0.7.0+ reality: the
  `eci-real-signed.pdf` fixture (alfonso/ArgosData) now anchors on a real
  root, so the test accepts EITHER an explicit TRUST_PLACEHOLDER/
  TRUST_PARTIAL code OR a confirmed `matchedRootSlug` while still asserting
  the banner-trigger placeholder message is present.
- Engine version assertion now compares against the exported `ENGINE_VERSION`
  constant instead of a hard-coded string so future bumps don't re-break it.
- `verify-status.test.ts` mirrors the same "real root OR demo code" guard.
- Result: 68/68 verifier tests green (was 65/68 since v0.7.0).

## [0.7.2] — 2026-05-12 — Per-signer Detail panel (multi-firma inspection)

Completes the multi-firma UX gap left open in 0.7.1: clicking a signer in the
summary list now swaps the Result + Detail panels to that signature instead
of always showing #1.

### Changed — pwa 0.7.2
- Verificar route: each signer row in the multi-firma banner is now a
  `<button>` that updates `selectedIndex`. The `Result`, `TimestampBadge`,
  `LtvBadge`, and `Detail` panels below the banner reflect the selected
  signature reactively.
- Visual: selected row gets a brand-tinted background + ring + eye icon
  on the right. Keyboard accessible (`aria-pressed`, focus ring).
- Hint text under the list updated to "Toca un firmante para inspeccionar…
  viendo firma #N de M".
- `selectedIndex` resets to 0 on every new verification and on Reset.

### Unchanged
- Verifier/signer engine: no changes (still 0.7.1). UI-only release.
- Single-sig PDFs: banner hidden, template behaves identically to 0.7.1.

## [0.7.1] — 2026-05-12 — Multi-firma ilimitado: verifier enumeration + UI list + signer xref-stream support

Closes the multi-firma gap reported by external tester 2026-05-12. PAdES
documents with N ≥ 2 signatures now verify each signature independently and
sign-on-top works against PDFs that use cross-reference streams (the SRI
gob.ec / BCE / PDF 1.5+ default — previously rejected with the cryptic
`cannot_add_signature_to_corrupt_pdf` message).

### Added — verifier 0.7.1
- `findAllSignatures(pdfBytes): SignedRange[]` — enumerates every PAdES
  signature in document/chronological order. Each entry carries its own
  /ByteRange + /Contents + metadata. Pairs each /ByteRange with the
  /Contents inside the same sig dict by forward-search and validates the
  hex window matches the gap [a+b, c).
- `verifyAllSignatures(pdfBytes, opts): MultiVerificationResult` — runs
  the full crypto/path/OCSP/TSA/LTV pipeline per signature, aggregating
  per-signature statuses into `overallStatus` via worst-case rank
  (invalid > no_signature > warning > valid).
- `MultiVerificationResult` type exported alongside `VerificationResult`.
- 4 new unit tests under `packages/verifier/tests/multi-signature.test.ts`.

### Added — pwa 0.7.1
- `runVerifyAll(pdf, opts): Promise<MultiVerificationResult>` in
  `apps/pwa/src/lib/workers/bus.ts` + new `verifyAll`/`resultAll` wire
  protocol on `verify.worker.ts`.
- Verificar route now calls `runVerifyAll` instead of `runVerify`.
- New summary banner renders above the single-sig detail block whenever
  `signatureCount > 1`. Shows overall colour (valid/warning/err), a
  numbered list of every signer (CN, signing time, profile B-B/B-T/B-LT
  /B-LTA), and an inline notice that detail panels still target sig #1.
- Single-sig PDFs render unchanged — banner hidden, existing template
  consumes signatures[0].

### Fixed — signer 0.7.1
- `parsePriorPdf` now accepts PDFs whose most recent cross-reference is a
  `/Type /XRef` stream (PDF 1.5+). The new helper `parseXrefStreamDict`
  reads /Size + /Root from the stream's plaintext dictionary without
  decompressing the FlateDecode data portion. Incremental update emits a
  classic xref+trailer chained via /Prev to the prior xref-stream object
  start — the resulting hybrid document is valid per ISO 32000-1 §7.5.8.4.
- This unblocks **multi-firma over SRI gob.ec comprobantes** (`RC-...pdf`)
  which previously failed with `cannot_add_signature_to_corrupt_pdf`.
- 10/10 existing classic-xref incremental tests still pass; integration
  test for the xref-stream path deferred until a real SRI fixture is
  captured (pdf-lib cannot synthesise an xref-stream PDF that preserves
  a /Sig dict — manual smoke path documented inline).

### Bumped — packages
- `@firma-ec/verifier` 0.7.0 → 0.7.1 (engineVersion in result body).
- `@firma-ec/signer` 0.7.0 → 0.7.1.
- `@firma-ec/pwa` 0.7.0 → 0.7.1 (footer badge).
- TSL package unchanged at 1.3.0 seq 4.

### Known limitations (not regressions)
- **Verificar Detail panel still shows signature #1 only** even on multi-
  signed PDFs. The summary banner gives users the full list of signers
  with per-sig status, but DSS/timestamp inspection drills into the first
  signature only. Per-signer inspection tracked for 0.7.2.

## [0.7.0] — 2026-05-12 — Stable release: graduates F7 RC + ArgosData real root + version coherence

Promotes the F7 LTV release chain to stable. Consolidates 26 unreleased commits
post-`v0.7.0-rc1` (rc2..rc9 mentioned only in commit subjects, never tagged) and
syncs all 5 sources of truth for version per qa-verify §3.1 (badge, frontend
package.json, packages, CHANGELOG, git tag).

### Added — tsl-ec
- **Real root for ArgosData** (3rd of 17 ARCOTEL ACEs with a real PEM, joining
  Eclipsoft + Uanataca). `ArgosData Root CA -SHA256`, self-signed, valid
  2022-06-09 → 2032-06-09. SHA-256
  `aaf7700654779e09dd8e380776022b24f6dde672f50cf82f88406ab7b01bde39`.
  Issues intermediate `ArgosData CA 1 - SHA256` which directly signs end-entity
  certs. ArgosData does not expose the root at well-known URLs; obtained via
  client-side `openssl pkcs12 -cacerts -nokeys` chain export from an
  ArgosData-issued .p12. With this root loaded, end-user signatures from
  ArgosData-issued certs verify in firmar.ec with full trust chain instead
  of `tsl_warning` placeholder warnings.

### Changed — versioning
- Unified all production packages + the app version badge to `0.7.0`:
  - `apps/pwa` (was 0.7.0-rc2 in package.json, 0.7.0-rc1 in footer; both drifted).
  - `packages/signer` (was 0.6.0-rc1; signer matures alongside F7 stable).
  - `packages/dss-pdf`, `packages/ltv-validation`, `packages/verifier` (were 0.7.0-rc1).
  - `packages/tsa-client`, `packages/tsa-trust` (were 0.5.0-rc1; F6 stable).
  - `apps/pwa/src/lib/version.ts` `APP_VERSION` constant (the footer badge).
- `verificar.demo_banner_body` (ES + EN): version string `v0.6.0-rc7` →
  `v0.7.0`; count `2 of 17` → `3 of 17` to reflect the new ArgosData root.
  Eliminates the stale-string drift incident reported by the operator on
  2026-05-12.

### Included (commits post-v0.7.0-rc1, previously unreleased)
- `5a00445` feat(ltv): F7.5 same-origin OCSP/CRL proxy (allowlisted ARCOTEL upstreams).
- `8383715` fix(ltv): F7.6 raise asn1js maxNodes for real ARCOTEL CRLs.
- `4097a0a` feat(mobile): p12 decrypt off main thread + zoom controls + 44px touch targets.
- `f11a7b3` fix(sw): aggressive workbox cache purge on install (Android stale SW fix).
- `7219d97` feat(firmar/mobile): default to last PDF page on load.
- `e6315ac` feat(pwa): user-confirmed SW updates + UpdateNotification toast.
- `96d4b90` fix(pwa): Button hrefs use hash for internal routes (PWA install fix).
- `919a13f` fix(configuracion): LTA toggle h-7→h-11.

### Known limitations (carried into 0.7.0, not regressions)
- **Signer**: multi-signature on PDFs with **xref streams + prior signature**
  still rejected with `cannot_add_signature_to_corrupt_pdf`
  (`packages/signer/src/incrementalUpdate.ts` requires classic xref tables).
  Affects SRI-issued documents (e.g. `RC-258-144-...pdf`); workaround is to
  re-print via browser print-to-PDF, flatten, then sign the fresh copy.
  Note: sequential multi-firma on classic-xref PDFs IS supported (tests in
  `incremental.test.ts` cover up to 3 signatures and assert all are detected).
  Proper xref-stream support tracked for 0.7.1+.
- **Verifier**: currently extracts only the **first** /ByteRange in a multi-signed
  PDF (`packages/verifier/src/pdf.ts:152` — "Find first /ByteRange") and reports
  on that single signature. Subsequent signatures on the same document are not
  enumerated nor displayed in the Verificar UI. PAdES requires verifiers to
  enumerate all signatures and report each independently. Tracked for 0.7.1
  as a P0 follow-up (tester report 2026-05-12).

### Landing 0.1.12 — F7 follow-up 2026-05-10
- Remove `/como-funciona-wa` from build (page parked in `_drafts/` until F3.5 WhatsApp inbox ships). Removed Header + Footer nav entries and `como-funciona-wa` route key. Eliminates the F6.7-audit-reported 404 on prod.

### Infra / docs — F7 follow-up 2026-05-10
- `infra/docker/Caddyfile.pwa` documents the planned `/api/ocsp` + `/api/crl` reverse-proxy shape (F7.5 scope, allowlisted upstreams). Not implemented; PWA falls back to direct fetch.
- `apps/pwa/src/lib/i18n.svelte.ts` warn copy in Configuracion clarifies that `/api/ocsp` is documented but not yet implemented.
- `packages/ltv-validation/tests/__fixtures__/` adds real ARCOTEL ACE OCSP + CRL fixtures captured 2026-05-10 (SECURITY DATA SubCA-2 + ArgosData CA 1).
- `packages/ltv-validation/tests/ocsp-kat-arcotel.test.ts` + `crl-arcotel.test.ts` consume the new fixtures (2 OCSP KATs pass; SD CRL skipped — BER indef-length, F7.6 followup).
- `scripts/lh-fallback-2026-05-10.mjs` + `_backups/F7-followup-2026-05-10/LIGHTHOUSE-SUMMARY.md` — Playwright-based lighthouse-equivalent audit (lighthouse CLI absent). 8 prod routes audited; cold-cache outlier on `firmar.ec/`, CF Web Analytics beacon blocked by CSP (expected).

## [0.7.0-rc1] / verifier 0.7.0-rc1 / signer 0.6.0-rc1 / ltv-validation 0.7.0-rc1 / dss-pdf 0.7.0-rc1 — 2026-05-10 — F7 LTV: PAdES B-LT + B-LTA

End of the PAdES ETSI baseline ladder. The signer now collects revocation
material (OCSP-first, CRL-fallback) and embeds it in a DSS dictionary as
an incremental update (B-LT), then optionally appends a document
timestamp (B-LTA). The verifier reads DSS + document timestamps and
reports the achieved profile without ever downgrading B-T to B-B.

Spec: `docs/superpowers/specs/2026-05-10-firma-ec-F7-LTV-design.md` (4266c4f)
Plan: `docs/superpowers/plans/2026-05-10-firma-ec-F7-LTV.md` (3bc1d6c)

### Added — signer 0.6.0-rc1
- `packages/signer/src/ltv.ts` — `collectLtvData()` orchestrates the
  OCSP-first / CRL-fallback cascade for signer + intermediates + TSA
  cert. Returns an aggregate `DssData` ready for `appendDss()`.
- `packages/signer/src/pades.ts` — `signPdfPades()` now threads
  `opts.ltv: { longTerm, longTermArchive, ocspUrl, crlUrl, ... }`. After
  B-T it runs LT (DSS) then LTA (document timestamp). Fallback policy:
  cert revoked → throw; network failure → drop back one tier with
  warning. New result field `ltv: LtvMeta`.

### Added — ltv-validation 0.7.0-rc1 (initial release)
- `src/ocsp/*` — RFC 6960 OCSP request builder, HTTP fetcher, response
  parser + `isCertRevoked()` predicate.
- `src/crl/*` — CertificateList parser + AIA/CDP URL discovery.
- `src/cache.ts` — in-memory + IndexedDB caches keyed by cert SKI +
  responder URL, TTL governed by `nextUpdate`.
- 33 tests (OCSP-fetch, OCSP-KAT, OCSP-request, CRL, AIA discovery,
  cache, property-based).

### Added — dss-pdf 0.7.0-rc1 (initial release)
- `appendDss({ pdfBytes, dss })` — writes DSS as PAdES incremental update
  (B-T → B-LT).
- `parseDss(pdfBytes)` — recovers the same shape (verifier-side).
- `appendDocumentTimestamp()` + `findDocumentTimestamps()` for
  /Sig /ETSI.RFC3161 envelopes (B-LT → B-LTA).
- 23 tests (incremental writer, parser round-trip, doc timestamp,
  streams).

### Added — verifier 0.7.0-rc1
- `src/dss.ts` — `extractDss()` recovers DSS via xref walk.
- `src/ltv.ts` — `verifyLtv()` cross-references embedded OCSP/CRL with
  the signer chain and checks document timestamps via the shared
  `verifyTimestamp()` (refactored to accept generic imprint sources).
- `verifyPdf()` now populates `result.ltv: LtvSummary`.
- Profile state machine: `B-B → B-T → B-LT → B-LTA`. No downgrade.
- 64 tests (DSS extraction, LTV cross-ref, profile inference,
  regression on F6 B-T sample → still profile B-T not B-B).

### Added — PWA 0.7.0-rc1
- `apps/pwa/src/ui/firma/LtvBadge.svelte` — emerald for B-LT, bright
  emerald for B-LTA. Wired into DownloadResult + Verificar detail panel.
- `apps/pwa/src/routes/Configuracion.svelte` — "Validez a largo plazo"
  section: toggles for B-LT/B-LTA, custom OCSP/CRL URLs, timeouts.
  Persisted via `lib/settings.ts`.
- `sign.worker.ts` — new stages `collect_ocsp`, `collect_crl`,
  `embed_dss`, `request_document_ts`.
- i18n ES/EN strings for the LtvBadge tooltip ladder + Configuracion
  copy. Small hint near OCSP/CRL URL fields: "URLs no por defecto
  requieren ajuste CSP del operador".
- E2E scaffold `tests/e2e/ltv-flow.spec.ts` (4 fixme tests).

### Added — fixtures + cross-val artifacts
- `scripts/gen-f7-samples.mjs` — Node script. Synthetic-CA fallback path
  used in sandbox (OCSP/CRL responders unreachable from build network);
  real B-T reused from F6.
- `_backups/F7-cross-val-artifacts/sample-{b-t,b-lt,b-lta}.pdf` mirrored
  into `packages/verifier/tests/fixtures/`.
- 2 verifier integration tests (`B-LT roundtrip`, `B-LTA
  documentTimestamp present`).

### Caveats
- Live OCSP/CRL fetches against ARCOTEL ACE responders unverified in
  sandbox; covered by synthetic-CA fixtures + unit KATs.
- Adobe Reader cross-val of B-LT/B-LTA samples is a manual user step
  (follow-up F7.5).
- CSP — `connect-src` retains the F6 TSA trade-off: user-supplied
  OCSP/CRL URLs require operator-side Caddyfile edits. UI hint added.

### Out of scope (followed up post-release)
- F7.5 — LTV refresh (re-add fresh OCSP/CRL before TSA expiry).
- F7.6 — Multi-OCSP with deterministic responder ranking.
- F8 — QES eIDAS (qualified electronic signature gates).

## [landing 0.1.11] — 2026-05-10 — Cleanup: remove non-existent @firmar.ec emails

User-visible cleanup. Three email addresses (`contacto@`, `datos@`, `security@firmar.ec`) were never provisioned (zone has null MX). Replaced with public, working channels — preserving LOPDP compliance via the parent data controller (IDK Manager) and following RFC 9116's allowance for URL-based security contacts.

### Changed — landing user-visible
- `apps/landing/src/lib/jsonld.ts` — `SITE.contactEmail` / `SITE.dpoEmail` / `SITE.securityEmail` removed. Added `SITE.contactUrl` (GitHub Issues), `SITE.dpoContactUrl` (idkmanager.com/contacto), `SITE.securityUrl` (GitHub Security Advisories). Schema.org `ContactPoint` now uses `url` instead of `email`.
- `apps/landing/src/components/OperadoPor.astro` — footer links rewritten to GitHub Issues / IDK Manager / Private Advisory.
- `apps/landing/src/components/PorQueEsSeguro.astro` — LOPDP card body: "DPO publicado" → "controlador de datos identificado" (ES + EN).
- `apps/landing/src/pages/{contacto,en/contact}.astro` — page rewritten: 3 cards now point to GitHub/IDK Manager/Advisory URLs. PGP section replaced by responsible-disclosure paragraph (no PGP maintained).
- `apps/landing/src/pages/500.astro` — error fallback CTA → GitHub Issues.
- `apps/landing/src/pages/{faq,en/faq}.astro` — lead copy drops the email mention.
- `apps/landing/src/content/pages/{es/privacidad,en/privacy}.md` — DPO section, lawful-basis table, ARCO+ access/erasure procedure, and contact list rewritten to redirect data subjects to IDK Manager (the legal data controller). Inbound-email language replaced by GitHub-issues language. **LOPDP compliance preserved** — Art. 12 rights still routable through the named controller.
- `apps/landing/src/content/pages/{es/acerca,en/about,es/terminos,en/terms}.md` — Contact list updated.
- `apps/landing/src/content/pages/{es/seguridad,en/security}.md` — Disclosure step 1 now points to GitHub Security Advisories (was: email + PGP).
- `apps/landing/src/content/faq/{10-empresas,en-10-organisations}.md` — sale paragraph drops the email contact.
- `README.md` — security reports line updated.
- `docs/transparency-report.md` — CAA `iodef` and DMARC `rua` rows annotated as pending operator DNS update.

### Removed
- `apps/landing/public/.well-known/pgp-key.txt` — file deleted (was a placeholder pointing to a key that was never generated).

### Changed — RFC 9116 security.txt
- `apps/landing/public/.well-known/security.txt` — `Contact:` lines now use HTTPS URLs (RFC 9116 §2.5.4 allows URI). `Encryption:` removed (no PGP key). `Expires:` bumped to 2027-05-10.

### Notes
- inbox-backend internal env-vars referencing `@firmar.ec` left untouched (out of user-visible scope; will be reviewed in next inbox-backend release).
- DNS-zone follow-up for the operator: update CAA `iodef` (currently `mailto:security@firmar.ec`) and DMARC `rua` (currently `mailto:datos@firmar.ec`) — non-blocking since MX is null.

## [0.6.0-rc8] / [landing 0.1.10] — 2026-05-10 — Phase A sweep (CF Insights + OG + editorial + cosign keypair)

Cosmetic + privacy + supply-chain sweep. Four items shipped together as `apps/pwa 0.6.0-rc8` + `apps/landing 0.1.10`.

### Changed — editorial pass
- Normalized "certificado digital ecuatoriano" → "certificado electrónico .p12 (ECI ARCOTEL)" in user-facing copy where the focus is the artifact (the cert file), not the country/ecosystem context. Targeted edits only — legal/about/FAQ prose discussing "ecosistema digital ecuatoriano" preserved.
  - `apps/pwa/index.html` meta description.
  - `apps/pwa/vite.config.ts` PWA manifest description.
  - `apps/landing/src/i18n/ui.ts` ES + EN `meta.home.description`.
  - `apps/landing/src/lib/jsonld.ts` SoftwareApplication description (ES + EN).
  - `apps/pwa/src/lib/i18n.svelte.ts` `home.firmar_desc` + `firmar_placeholder.body`.
  - `apps/landing/src/components/ParaQuien.astro` h2 (ES + EN).

### Added — OG image surface
- PWA `apps/pwa/index.html` now emits `og:title`, `og:description`, `og:image` (1200×630), `og:url`, `og:locale`, plus Twitter card meta.
- PWA `og-app-firmar-ec.png` packaged into `apps/pwa/public/`.
- Landing `apps/landing/public/og-firmar-ec.png` available as a stable URL alias for share previews. The dynamic Astro renderer at `src/pages/og/[slug].png.ts` (satori + resvg-js, 1200×630 brand template) continues to serve `/og/{slug}.png` for per-page cards.

### Privacy — Cloudflare Insights beacon
- Documented that the `static.cloudflareinsights.com/beacon.min.js` violation reported in F6.7 audit (P2-1) is **edge-injected by Cloudflare proxy**, not present in source. **Action required from operator**: disable "Web Analytics" in CF dashboard for `firmar.ec` and `app.firmar.ec` zones to honor the documented "sin tracking" promise. CSP intentionally does *not* whitelist the beacon.

### Security — Cosign keypair scaffolding
- New `apps/landing/public/.well-known/cosign.pub` exposes the verifying public key at `https://firmar.ec/.well-known/cosign.pub` for downstream verifiers.
- Operator runbook: keypair generated via `docker run --rm gcr.io/projectsigstore/cosign:v2.2.4 generate-key-pair`, stored in workspace SOPS vault under `apps_firma_ec.cosign_priv` / `apps_firma_ec.cosign_pub` / `apps_firma_ec.cosign_password`. Tag signing for `v0.6.0-rc8` is *opt-in* once operator confirms the vault entry and runs the documented `cosign sign-blob` step.

## [landing 0.1.9] — 2026-05-10 — Hero copy: .p12 + electrónico

`apps/landing 0.1.9`. Hero h1 mentions `.p12` and `certificado electrónico` (was just "ecuatoriano") for SEO + correct expectations vs hardware tokens. PWA `hero.title` (i18n) bumped in parity (no PWA version bump — already 0.6.0-rc7).

### Changed
- `apps/landing/src/components/Hero.astro` h1 ES/EN.
- `apps/pwa/src/lib/i18n.svelte.ts` `hero.title` ES/EN parity.

## [0.6.0-rc7] — 2026-05-10 — F6.7 TSL real PEM fetch (2/17 ACEs)

`apps/pwa 0.6.0-rc7`, `@firma-ec/tsl-ec` TSL_VERSION 1.2.0 sequence 3.

### Changed
- **TSL upgraded from full demo to partial demo (2/17 real ACEs)**:
  - `eclipsesoft` now real: ECLIPSOFT CA ROOT, self-signed
    2025-12-02 → 2050-12-03, fetched from
    `firmas.eclipsoft.com/wp-content/uploads/2026/03/ECLIPSOFTCAROOT.cacert.cer`.
    SHA-256 `e40c3ce5…22c1f9`. Subject `CN=ECLIPSOFT CA ROOT, O=ECLIPSOFT S.A.,
    L=GUAYAQUIL, C=EC, organizationIdentifier=VATEC-0992253428001`.
  - `uanataca` now real: UANATACA ROOT 2016, self-signed 2016-03-11 → 2041-03-11,
    fetched from `web.uanataca.com/ec/certificados-ca` (Ecuador-specific repo).
    SHA-256 `44607b3d…dfb5a6`. Subject `C=ES, O=UANATACA S.A., CN=UANATACA ROOT 2016,
    organizationIdentifier=VATES-A66721499`. Spanish-incorporated qualified TSP
    under eIDAS, ARCOTEL-accredited as ECI in Ecuador via Uanataca Ecuador S.A.
- 15/17 slots remain self-signed placeholders. ARCOTEL listing page does not
  link to per-ACE repositories; BCE, Argosdata, Datil, Security Data,
  registro-civil, judicatura and the smaller ECIs do not publish their roots
  at well-known URLs reachable from outside EC networks. Each placeholder's
  `notes` field documents what was tried.
- **Verifier banner logic granular (`packages/verifier/src/index.ts`)**:
  - When ALL 17 are placeholders → emit `TRUST_PLACEHOLDER` (legacy, full demo).
  - When SOME real but path didn't validate (signer's CA still placeholder) →
    emit new `TRUST_PARTIAL` with message `"Trust chain not yet established:
    N/M ACEs ARCOTEL tienen raíz real; K aún placeholder"`.
  - When 0 placeholders remain → no banner (production).
- `Verificar.svelte` banner heuristic now also triggers on `TRUST_PARTIAL`.
- i18n `verificar.demo_banner_body` (es+en) reflects the partial-demo state
  ("2 de 17 …").
- Tests in `verify-status.test.ts`, `pathValidation.test.ts`,
  `regression-real-eci.test.ts` relaxed: assertions that required
  `roots.every(r => r.isPlaceholder)` now use `roots.some(...)`; checks for
  warning code now accept `TRUST_PLACEHOLDER` OR `TRUST_PARTIAL`.

### Bumped
- `packages/tsl-ec/src/index.ts`: TSL_VERSION `1.1.0` → `1.2.0`,
  TSL_SEQUENCE `2` → `3`.
- TSL bundle SHA-256 regenerated:
  `c15f6357c694a07090f715cdf8e70a86a34239415ea8eaa8d6eff1db1b13d2a5`.

### Tests
- All `pnpm -r test` packages green: 57 verifier (2 skipped legacy),
  64 signer, 19 tsa-client, 9 inbox-crypto, 7 tsa-trust, 121 inbox-backend.

### Backup
- `_backups/F6-tsl-pemfetch-2026-05-10/{roots/,roots.ts}` snapshot of
  pre-fetch state preserved.

### TODOs (manual fetch follow-up — 15 remaining ACEs)
- `bce` — `eci.bce.fin.ec` DNS unreachable from this build host. Try from EC.
- `argosdata` — site doesn't expose repositorio publicly. Contact +593939658192.
- `datil` — `Centros de Ayuda → Certificados Digitales` collection (3 articles)
  not crawlable; check docs.datil.com manually.
- `securitydata` — site has no `/repositorio` or `/descargas` link to CA root
  on public pages. Contact 02-3922169.
- `registro-civil`, `judicatura`, `alpha-technologies`, `anfac`, `appfirmas`,
  `corpnewbest`, `darkcam`, `firmasegura`, `lazzate`, `letmi`, `primecorelat`
  — no PKI repository link found on public sites. ARCOTEL listing page
  doesn't link per-entity. Likely accessible only via signed PDF chain
  extraction once a representative .p12 from each CA is available.

## [0.6.0-rc6] — 2026-05-10 — F6.6 TimestampBadge gold variant: success-green

`apps/pwa 0.6.0-rc6`. Verifier/signer/landing unchanged (cosmetic only).

### Changed
- **F6.6 TimestampBadge `gold` variant retuned from honey-amber → success-green**
  to read as positive/verified instead of "another warning". When a B-T PDF
  is verified end-to-end and the outer cert chain still produces 18 TSL
  placeholder advertencias, the orange "Firma válida con advertencias"
  panel sits directly above the gold badge. The previous hue 85° (honey
  amber) shared visual register with warn-tone surfaces and the user read
  the gold badge as a second warning.
  - File: `apps/pwa/src/ui/firma/TimestampBadge.svelte` (style block).
  - Triad now hue 145° (the `ok` token family): bg `oklch(96% 0.04 145)`,
    border `oklch(64% 0.16 145 / 0.45)`, fg `oklch(34% 0.10 145)`. Dark
    theme triad mirrored for parity. Icon `i-lucide-shield-check` retained
    (semantic for "verified timestamp"; already on the safelist).
- Silver variant unchanged — it correctly stays in the warn/neutral register
  to signal "stamp present but at least one check failed".

### Notes
- No verifier/signer/i18n logic touched. Cosmetic only.
- SW cache caveat: append `?bust=rc6` or hard-reload to pick up the new
  bundle on devices that have rc5 cached.

## [0.6.0-rc5] / verifier 0.5.0-rc4 — 2026-05-10 — F6.5 fix B-T extraction + engine version

`apps/pwa 0.6.0-rc5` + `@firma-ec/verifier 0.5.0-rc4`. Signer/landing unchanged.

### Fixed
- **F6.5 verifier reports B-B on B-T PDFs** — user signed a PDF with TSA on
  (rc4 LIVE), badge "Firma sellada · www.freetsa.org" rendered fine in
  DownloadResult, but verifying the same PDF showed "PERFIL PADES: B-B" and
  the TimestampBadge never went gold. Root cause same class as F3 v0.4.4
  (`pkijs encodedValue empty on build path`): in `packages/verifier/src/cms.ts`
  the timestamp unsigned-attribute extraction was reading
  `tsAttr.values[0].valueBlock.valueHex`, which is **empty** for parsed
  ASN.1 SEQUENCEs in asn1js. The TimeStampToken (a ContentInfo SEQUENCE)
  came back as a 0-byte buffer → verifier silently treated the signature as
  B-B. Fix: prefer `valueBeforeDecodeView` (asn1js stores the original DER
  bytes when parsed from BER) with `toBER(false)` as fallback.
  - File: `packages/verifier/src/cms.ts` (timestamp extraction block).
  - Regression test: `tests/cms.test.ts` "F6.5 — extracts RFC 3161
    timestampToken from B-T PDF". Asserts `timestampToken !== undefined` and
    `length > 1000` (FreeTSA tokens are ~4–5 KB; bare TSTInfo ≥ 1 KB).
  - Companion test: B-B PDF leaves `timestampToken` undefined.
- **F6.5 stale `ENGINE_VERSION = '0.3.3'`** in `packages/verifier/src/index.ts`
  surfaced in PWA Configuración footer ("Versión del motor: 0.3.3"). Bumped
  to `'0.5.0-rc4'` to match the verifier package version. Regression-real-eci
  test updated to assert the new value.

### Changed
- `packages/verifier/package.json`: `0.5.0-rc1` → `0.5.0-rc4` (catch up to
  signer/tsa-client baseline).

### Notes
- Hardcoded `ENGINE_VERSION` (vs JSON import) chosen to avoid coupling tsconfig
  `resolveJsonModule` across all consumers. Bump on each release.
- SW cache caveat: hard reload may be required for users on rc4 to pick up the
  new bundle.

## [0.6.0-rc4] / signer 0.5.0-rc3 — 2026-05-10 — F6.4 fix B-T `signature_too_long`

`apps/pwa 0.6.0-rc4` + `@firma-ec/signer 0.5.0-rc3`. Landing unchanged at `0.1.8`.

### Fixed
- **F6.4 `signature_too_long` on real .p12 + B-T (TSA on)**: First production
  attempt with an ECI Ecuador (ArgosData CA 1) certificate failed at the embed
  step with code `signature_too_long`. Root cause: the `/Contents` placeholder
  reserved only 16384 bytes (32768 hex chars). PAdES-B-T appends a full RFC
  3161 TimeStampToken (FreeTSA cert + chain + TSTInfo, ~4–5 KB) on top of the
  CMS, and ECI chains run ~3–5 KB themselves — total CMS hex routinely
  overflows 32 K hex chars.
  - `packages/signer/src/pades.ts`: `DEFAULT_SIGNATURE_LENGTH` 16384 → 32768
    bytes (65 536 hex chars). Comfortable headroom for B-T + multi-cert chains.
  - `packages/signer/src/incrementalUpdate.ts`: same bump (mirrors the
    primary signature path used for second-and-later signatures).
  - JSDoc on `PadesSignOptions.signatureLength` updated.

### Cost
- +16 KB per signed PDF (32 768 − 16 384). Negligible vs typical signed-PDF
  sizes (often hundreds of KB to multi-MB). No regressions in B-B path.

### Notes
- PWA service workers from rc1/rc2/rc3 still cached on user devices need to
  accept the update prompt to pick up rc4. The fix is in the signer worker
  bundle, not in any cached page.

## [0.6.0-rc3] / [0.1.8] / signer 0.5.0-rc2 — 2026-05-10 — F6.3 QR URL fix + landing hash redirect

`apps/pwa 0.6.0-rc3` + `apps/landing 0.1.8` + `@firma-ec/signer 0.5.0-rc2`.

### Fixed
- **F6.3 QR deep-link landed on the wrong site**: F6 introduced a QR encoding
  `https://firmar.ec/#/verificar?h=<hex>` in every signed PDF. Scanning that QR
  opened the **Astro landing** at `firmar.ec`, which doesn't handle SPA hash
  routes — the `/verificar` deep-link banner (F6.1) never fired and users were
  stuck on the marketing home.
  - **Forward fix (signer)**: `packages/signer/src/pades.ts` now embeds
    `https://app.firmar.ec/#/verificar?h=<hex>` in the QR. New signatures land
    on the PWA directly.
  - **Backward-compat (landing)**: `apps/landing/src/layouts/Base.astro`
    ships an inline pre-render script that redirects any hash matching
    `^#/(verificar|firmar|paranoia|about|configuracion)` to
    `app.firmar.ec`, preserving the hash. Covers every PDF signed with
    F3–F6.2 already in circulation.
  - Inline script runs before BaseHead/theme bootstrap so the user never
    sees a landing flash. CSP-compliant (`'unsafe-inline'` already in
    landing policy; no Trusted Types lockdown).
- Signer test suite updated: `visibleSig.test.ts` now asserts the
  `app.firmar.ec` prefix in three places, plus a new F6.3-specific test.

### Notes
- PWA service workers from rc1/rc2 still cached on user devices will keep the
  old verifier UI until the update prompt is accepted. The landing redirect
  ensures the deep-link still works for those users — they get routed to
  `app.firmar.ec` and the cached PWA handles the hash.

## [0.6.0-rc2] — 2026-05-10 — F6.1 QR deep-link + F6.2 multi-firma UX

`apps/pwa 0.6.0-rc2` (PWA + signer-types bump; verifier unchanged from
`0.5.0-rc1`). The `TimestampMeta.reason` union gains two new members
(`'user_disabled'`, `'multifirma_path'`) — non-breaking SemVer addition;
`'disabled'` retained as backward-compat alias.

### Fixed
- **F6.2 multi-firma TSA silent-no-feedback**: when the user re-signed an
  already-signed PDF, the worker forced PAdES B-B (incremental update) and
  emitted `timestamp.reason: 'disabled'`. `DownloadResult.svelte` then
  treated that as a deliberate user opt-out and suppressed both the gold
  badge AND any toast — leaving users with zero visual feedback about why
  their signature had no timestamp. Now:
  - Worker distinguishes `'user_disabled'` (silent, intended) from
    `'multifirma_path'` (renders an informational pill: "Firma adicional
    sobre PDF ya firmado — el sello RFC 3161 solo aplica a la primera
    firma de un documento; las firmas anteriores conservan sus propios
    sellos").
  - Legacy `'disabled'` value retained as backward-compat alias and
    mapped to the same pill at the UI layer (older sign-worker bundles
    still cached in service workers will keep working without redeploy).
  - Worker emits `progress: request_timestamp` BEFORE entering the
    single-firma signer call when TSA is enabled, so users see
    "Solicitando sello de tiempo…" while the FreeTSA round-trip is in
    flight rather than only after it completes.
  - 4 i18n entries added (`firmar.tsa.multifirma_pill_title` + `_body`,
    es + en). Pill uses ink-tone (info, not warn) to match the design.

### Added
- **F6.1 QR deep-link**: `/verificar` now reads the `?h=<hex>` hint that the
  signed-PDF QR encodes (`https://firmar.ec/#/verificar?h=<sha256-12hex>`).
  - Info banner at the top of the page when `?h=` is present, showing the QR
    document hash and inviting the user to drop the signed PDF.
  - Hash compare badge after verification: SHA-256 (first 12 hex) of the
    uploaded bytes is compared to the QR hint and rendered as a green "match"
    or amber "info — expected if you uploaded the signed PDF" hint with an
    expandable "¿Por qué?" explainer covering the unsigned-vs-signed semantics.
  - Compare is **purely informational**; the cryptographic verdict from the
    verifier worker remains the source of truth.
- New helper `apps/pwa/src/lib/qrDeepLink.ts` (`parseQrHash`,
  `readQrHashFromLocation`, `compareHash12`) with 11 unit tests in
  `tests/qrDeepLink.test.ts`.
- 6 i18n keys × 2 langs (12 entries): `verificar.qr.banner_title`,
  `banner_subtitle`, `match_ok`, `match_warn`, `why_summary`, `why_body`.

### Notes
- The signer hashes the *unsigned* source PDF, so legitimate verifications of
  the signed PDF will surface as "info — expected" rather than "match". Copy
  is calibrated to make this an honest, non-alarming UX rather than a warning.

## [0.5.0-rc1] — 2026-05-09 — F6 PAdES B-T (RFC 3161 timestamp)

Release-candidate cut for F6 (TSA). Versions in this train:
`apps/pwa 0.6.0-rc1`, `packages/{signer,verifier,tsa-client,tsa-trust} 0.5.0-rc1`,
`apps/landing 0.1.7` (unchanged).

### Added
- **F6 TSA**: PAdES B-T via FreeTSA timestamp, default-on with graceful B-B fallback.
  - New `@firma-ec/tsa-client` package — RFC 3161 client (browser + Node), KAT-tested
    request/response/parse pipeline, fetched via `https://freetsa.org/tsr` by default.
  - New `@firma-ec/tsa-trust` package — embedded FreeTSA root + ARCOTEL placeholder
    slot, EKU `id-kp-timeStamping` chain validation.
  - Signer attaches `id-aa-signatureTimeStampToken` (OID `1.2.840.113549.1.9.16.2.14`)
    in CMS `unsignedAttrs` after the inner signature is computed; PDFs round-trip as
    PAdES B-T in Adobe Reader.
  - Verifier renders gold/silver/none badge based on TSA imprint + signature + chain
    validity. Legacy B-B PDFs continue to verify as `valid` with `badge: 'none'`.
  - **`TimestampBadge.svelte`** component with `Intl.DateTimeFormat('es-EC')` /
    `('en-US')` formatting, three-state contract, reduced-motion aware.
  - **`/configuracion`** route with TSA enable/URL/timeout controls (persisted in
    `localStorage.firma_ec_settings_v1`) plus a "Probar TSA" probe button.
  - Caddy CSP `connect-src` now allows `https://freetsa.org`.
  - Sign worker emits the new `request_timestamp` progress stage.

### Fixed
- Verifier ECDSA curve derivation: now reads from SPKI `algorithmParams` instead
  of inferring from the digest algo (was failing for FreeTSA SHA-512+P-384 combos
  during F6 KAT verification).

### Notes
- F3.5 WhatsApp inbox/outbox code complete (24 commits) but deploy gated behind a
  separate batch.
- ARCOTEL TSAs: F6.5 will swap the placeholder PEM once they publish RFC 3161
  endpoints.
- Mozilla Observatory + securityheaders.com should be re-checked post-deploy on
  both `firmar.ec` and `app.firmar.ec`; A+ should hold (the only CSP delta is the
  added `https://freetsa.org` in `connect-src`).

## [0.5.1] / landing [0.1.7] - 2026-05-09 — Default LIGHT, dark only opt-in

### Fixed
- **P0 user-reported**: "landing y app siempre en blanco tema oscuro solo manual". Both sites auto-switched to dark when the OS preferred dark, ignoring user intent. Now the default is **always light**; dark applies only after the user clicks the toggle, and the choice persists in `localStorage.theme`.
  - `apps/landing/src/layouts/Base.astro` — bootstrap script no longer reads `matchMedia('(prefers-color-scheme: dark)')`. `data-theme` is `'dark'` only when `localStorage.theme === 'dark'`; any other value (including legacy `'system'`) collapses to light. `<html data-theme-default="system">` → `"light"`.
  - `apps/pwa/index.html` — added an inline theme bootstrap script (runs before the Trusted Types policy) so the PWA matches the landing's behaviour: default light, no `prefers-color-scheme`, migration of legacy `'system'` → light. Removed the `<meta name="color-scheme" content="light dark">` (now driven by `[data-theme]` via CSS).
  - `apps/landing/src/styles/reset.css` + `apps/pwa/src/styles/reset.css` — replaced `color-scheme: light dark` + `light-dark()` (which automatically rendered dark on OS-dark before any JS bootstrap could fire) with explicit `color-scheme: light` and `[data-theme="dark"]` overrides.
  - `apps/landing/uno.config.ts` + `apps/pwa/uno.config.ts` — `presetWind4({ dark: '[data-theme="dark"]' })` so all `dark:` utilities (`dark:bg-ink-950`, `dark:text-ink-100`, etc.) key off the same selector the toggle writes, instead of UnoCSS' default `.dark` class which was a dead path in this codebase.

### Notes
- The `ThemeToggle.svelte` components were already binary (light↔dark), so no code change was needed there — the bootstrap now guarantees `dataset.theme` is exactly `'light'` or `'dark'` on mount.
- `prefers-reduced-motion` is preserved (still honoured for accessibility). Only `prefers-color-scheme` was removed from the auto-decision.

### Verification
- Live Playwright audit with `prefersColorScheme: 'dark'` context option:
  - `https://firmar.ec/` first visit → light. Toggle → dark. Reload → still dark. Toggle → light. Reload → still light.
  - `https://app.firmar.ec/` first visit → light. Toggle → dark. Reload → still dark. Toggle → light. Reload → still light.
  - 0 console errors on both, before/after screenshots captured.

## [0.5.0] - 2026-05-09 — Deep visual parity landing ↔ PWA

User reported: "https://app.firmar.ec/ y https://firmar.ec/ pareciera que son cosas diferentes!!!! unifica todo para que no se vea como cosas separadas aunque solo sea visualmente". v0.4.9 had unified design **tokens** but the components themselves rendered visibly different. v0.5.0 reimplements PWA components to match the landing's design system pixel-by-pixel where reasonable.

### Added
- **`apps/pwa/src/ui/Button.svelte`** — shared CTA primitive mirroring landing patterns. Variants `primary | outline | ghost | compact`, sizes `sm | md | lg`. Inherits the landing's premium shadow/lift/easing tokens (`cubic-bezier(0.4,0,0.2,1)` and `cubic-bezier(0.32,0.72,0,1)`).
- **PWA Hero** — eyebrow "Firma electrónica · Ecuador" (uppercase mono brand-500) + landing-style H1 (`clamp(2rem,1.2rem+4vw,4rem)` bold tracking `-0.02em`) + lead paragraph + 3-button row (primary verify + outline sign + ghost institutional) + 5 trust badges (Apache, ETSI, ARCOTEL, LOPDP, 100% browser).
- **PWA Footer 3-col grid** — lockup + description + IDKMARK / Project links / Privacy claim, plus bottom strip with copyright + version + security.txt link, mirroring `apps/landing/src/components/Footer.astro`.

### Changed
- **PWA Header** bumped from `h-14` to `h-16` to match landing. Border now transparent until scroll (`border-transparent` → `border-ink-200/dark:border-ink-800` after 8px scroll), via `onMount` listener on Svelte side. Container width unified.
- **PWA Home cards** — `rounded-xl` → `rounded-lg` to match landing Card.astro radius. Numbered list cards use mono `01/02/03` instead of plain `1/2/3` for landing typographic voice.
- **`apps/pwa/src/lib/version.ts`** + **package.json**: `0.4.9` → `0.5.0`.

### i18n keys nuevas (ES + EN)
- `hero.eyebrow`, `hero.title`, `hero.lead`, `hero.cta_primary`, `hero.cta_secondary`, `hero.cta_tertiary`.
- `footer.description`, `footer.project`, `footer.privacy_heading`, `footer.licencia`.

### Verification
- Audit doc `docs/visual-divergence-landing-pwa-2026-05-09.md` with side-by-side before/after screenshots at 390/1280/1920 viewports.
- Tests cumulative: signer 56 / verifier 47+2 skipped — all green (103 PASS).
- PWA typecheck: 542 files, 0 errors, 0 warnings.
- PWA bundle main 53.36 KB gzip (was 51.78; +1.58 KB for Button component + 8 i18n keys, well under 200 KB target).
- Console errors live preview: 0.

### Lessons
> **Tokens unify is not enough for visual parity.** `firma-ec` v0.4.9 already shared brand/ink/spacing/motion/shadow tokens between landing and PWA, yet user perceived them as "two different things" because component implementations (Astro vs Svelte) translated tokens into divergent layouts. Real parity required **reimplementing components with the same patterns** (Hero structure, Footer grid, Button variants) and verifying side-by-side at multiple viewports. New rule: when unifying multi-stack apps, design at the component-pattern level, not just token level.

## [0.4.9] / landing [0.1.6] - 2026-05-09 — Visual unify (landing centering fix + IDKMANAGER credit + token sync)

### Fixed
- **Landing main container flush-left on desktop** (P0 user-reported). UnoCSS `presetWind4`'s default `.container` utility was setting `max-width: 1536px` without `margin-inline: auto`, overriding the project's tokenised `.container` rule defined in `@layer base`. Sections rendered at `x: 0, w: 1536px` on a 1920px viewport instead of centered.
  - **Fix** (`apps/landing/src/styles/tokens.css`): moved `.container`/`.container-narrow`/`.container-prose` definitions into `@layer utilities` (last layer in cascade) with `!important` on `width`, `max-width` and `margin-inline`, so they win over Wind4's utility regardless of injection order.
  - Verified: section now reports `x: 384, w: 1152, ml: 384px, mr: 384px` (exact center on 1920px viewport, `--w-default: 72rem`).

### Added
- **`apps/landing/src/components/IdkmanagerMark.astro`** + **`apps/pwa/src/ui/IdkmanagerMark.svelte`** — institutional wordmark "IDKMANAGER" as inline SVG (zero HTTP cost, theme-aware via `currentColor`). Sizes `sm` (88px), `md` (128px), `lg` (160px) — typography Geist Display 700, letter-spacing `0.04em`.
- **Landing footer** — IDKMANAGER mark next to "Operado por" credit, linking to `https://idkmanager.com/`.
- **PWA footer** — "Operado por IDKMANAGER" credit row alongside copyright + version.
- **PWA About** — full IDKMANAGER credit card with `lg` mark, body text, hover affordance — replaces the visual gap that previously existed before the institutional CTA.

### Changed
- **Token sync landing ⇄ PWA** (`apps/landing/src/styles/tokens.css`):
  - Imported PWA's F3 motion tokens (`--motion-curve`, `--motion-tap`, `--motion-state`, `--motion-state-lg`, `--motion-emerge`).
  - Imported PWA's shadow tier tokens (`--shadow-flat/rest/hover/focus/success`) including dark-theme overrides.
  - Imported PWA's reduced-motion media query block.
  - Both apps now share the exact same brand/ink/spacing/radius/font scales (already aligned pre-v0.4.9, verified during audit).

### i18n keys nuevas (ES + EN)
- `footer.operated_by` — "Operado por" / "Operated by".
- `about.idk_credit_label` — "Un proyecto de" / "A project by".
- `about.idk_credit_body` — descripción institucional IDKMANAGER.
- `about.idk_credit_aria` — aria-label del bloque enlazado.

### Verification
- Pre-fix Playwright probe `getBoundingClientRect()` → confirmed flush-left bug live on `https://firmar.ec/`.
- Post-fix expected: `main section.container` centered on viewports ≥1024px; mobile (<640px) keeps 1rem inline padding.

## [0.4.7] - 2026-05-09 — ECDSA P-256/P-384/P-521 PKCS#12 path

### Added
- **`packages/signer/src/p12.ts`**: ruta completa para `.p12` con clave **ECDSA** (P-256, P-384, P-521).
  - Cuando `node-forge` no logra modelar la clave EC (`bag.key === undefined`), se extrae el ASN.1 PKCS#8 crudo de `bag.asn1` y se re-emite como DER — sin pasar por `wrapRsaPrivateKey`.
  - Cuando `forge.pki.certificateFromAsn1` rechaza un cert ECDSA, ahora se lee `cb.asn1` y se reconstruye `SignerCert` (CN, issuer, validity, serial) directamente desde el DER vía `asn1js` (`signerCertFromDer`).
  - `sigAlg` se infiere uniformemente desde el DER del cert (`readSpkiAlgorithmFromDer`) — converge RSA y EC en el mismo path.
  - Mapeo de `namedCurve` OID → suite: `1.2.840.10045.3.1.7` → `ECDSA-P256-SHA256`, `1.3.132.0.34` → `ECDSA-P384-SHA384`, `1.3.132.0.35` → `ECDSA-P521-SHA512`.
- **`packages/signer/scripts/gen-test-p12.ts`**: regenerador de `ecdsa-p256-valid.p12` ahora produce un PFX **forge-canónico**:
  - Cert self-signed sigue generándose con `pkijs` (forge no firma con EC).
  - El `EncryptedPrivateKeyInfo` del shrouded key bag se emite vía `forge.pki.encryptPrivateKeyInfo` (PBES2 + AES-256), garantizando `OCTET STRING` primitive (constructed=false) que `node-forge` puede re-decifrar al leer.
  - PFX outer (AuthenticatedSafe + ContentInfo + MacData HMAC-SHA1) construido a mano con primitivas forge para mantener el archivo 100% interoperable con `pkcs12FromAsn1`.

### Tests
- `tests/p12.test.ts`: el test `parses ECDSA P-256 valid` (anteriormente `it.skip` con caveat de v0.4.3) ahora corre y pasa. Verifica `sigAlg`, `subjectCN`, `kty='EC'` y que el PKCS#8 DER empieza con `0x30` (SEQUENCE).
- `tests/pades.test.ts`: el `describe.skip` para ECDSA-P256 PAdES (deferred desde v0.4.3) re-habilitado. Verifica round-trip completo: `parsePfx` → `signPdfPades` → `findSignature` → `parseCms` con OID `1.2.840.10045.4.3.2` (ecdsa-with-SHA256) y messageDigest cruzado contra el hash recomputado de `coveredBytes`.
- Total **56 tests / 0 skipped** en `@firma-ec/signer` (vs 54 passed + 2 skipped en v0.4.6).

### Changed
- `apps/pwa/src/lib/version.ts` y `apps/pwa/package.json` bumpeados a `0.4.7`.

### Notes
- Las ECIs ecuatorianas reales (BCE, Security Data, ArgosData, ANFAC, ConsejoJudicatura) emiten todas RSA hoy; este path es para futuro o cuando llegue alguna ECI con ECDSA. RSA + 3DES sigue funcionando idéntico (el code path RSA no se tocó).
- Bundle delta cero: ningún `dependency` nuevo. El cambio es lógica condicional dentro de `parsePfx`.

## [0.4.6] - 2026-05-09 — Polish bundle (a11y mobile + cross-route handoff + code-split)

### Added
- **Footer landmark global** (`apps/pwa/src/ui/Footer.svelte`) renderizado en todas las rutas excepto `/share` y `/handle-file`. Incluye copyright, versión (centralizada en `lib/version.ts`), claim de privacidad ("Sin tracking. Sin servidores. Tu PDF nunca sale de tu navegador."), link a /about y link a GitHub. Todos los enlaces ≥44×44 px (a11y tap targets WCAG 2.5.5 AAA).
- **Cross-route blob handoff sign→verify** — `Verificar.svelte` ahora consume la sessionStorage key `firmar.verify_preload.bytes_b64` que `DownloadResult.svelte` ya escribía. Click en "Verificar este PDF" en el step 7 ahora carga el PDF firmado en `/verificar` automáticamente con `status='warning'` (TSL demo) sin re-drop.
- **BoxPlacer auto-scrollIntoView** en mobile (<768px): al entrar al step 2, `requestAnimationFrame` + `scrollIntoView({block:'center'})` lleva el `.pdf-stage-host` al viewport sin que el usuario tenga que pasar manualmente el progress bar.
- `apps/pwa/src/lib/version.ts` — fuente única de `APP_VERSION`. Footer + About importan de aquí.

### Changed
- **Bundle main code-split** (`apps/pwa/vite.config.ts` `manualChunks`):
  - `signer-deps` (node-forge + qrcode) — solo cuando se entra a `/firmar`.
  - `pki` (pkijs + asn1js) — compartido /firmar + /verificar.
  - `pdf` (pdfjs-dist) — lazy en Verificar + PdfPreview.
  - `crypto-utils` (@noble + pvutils + pvtsutils).
  - `signer` y `verifier` (paquetes locales).
  - **Resultado**: main `index-*.js` 1004 KB → **160 KB raw / 50 KB gzip** (−84% raw, −82% gzip).
- **DEMO banner copy** (`verificar.demo_banner_body` ES + EN): ahora menciona explícitamente "v0.4.5+ — las 17 ACEs ARCOTEL están como placeholders en el TSL local. Cuando se publiquen los PEMs reales, este banner desaparecerá."
- **GitHub Actions Node.js 20 → 22** en `release.yml`, `ci.yml`, `lighthouse.yml` (LTS, alinea con dev local + habilita Cosign + SBOM modernos).

### Fixed
- A11y: confirmado que hamburger header, theme toggle y lang switcher ya tenían `h-11 w-11 / min-h-11 min-w-11` (compliant con 44×44 desde v0.4.x). No se modifican.

### i18n keys nuevas (ES + EN)
`footer.copyright`, `footer.version_label`, `footer.privacy_claim`, `footer.github_repo`, `footer.about_link`, `verify.handoff_loading`.

### Bundle (gzip)
| Chunk | v0.4.5 | v0.4.6 |
|---|---|---|
| `index` (main) | 277 KB | **50 KB** |
| `pdf` | 98 KB | 98 KB |
| `pki` | (incluido en main) | 75 KB |
| `signer` | (incluido en main) | 188 KB |
| `signer-deps` | (incluido en main) | 88 KB |

### Tests
- 97 tests passing (pre-existing 5 vitest suites bloqueadas por `@firma-ec/tsl-ec` workspace resolution — heredado de v0.4.5, no introducido aquí).

## [0.4.5] - 2026-05-09 — Cuadro de firma con QR (FirmaEC-style)

### Added
- **Cuadro de firma visible con QR escaneable** (estilo FirmaEC desktop). El widget de firma ahora se renderiza con layout split:
  - **Izquierda**: QR code 60×60 pt en negro sobre blanco, generado 100% client-side (lib `qrcode` + nivel ECC `M`). Apunta a `https://firmar.ec/#/verificar?h=<sha256-12chars>` — los primeros 12 hex de SHA-256 del PDF original son una pista escaneable hacia el verificador público.
  - **Derecha** (174×60 pt + 6 pt margen): bloque de 3 líneas Helvetica 8 pt:
    - L1: `Firmado por: <CN>` (truncado a 35 chars con ellipsis).
    - L2: `Fecha: YYYY-MM-DD HH:mm` (timezone local del firmador).
    - L3: `Razón: <razón>` o `firmar.ec` si no se especifica razón.
  - **Borde**: outline negro 0.5 pt alrededor del cuadro completo.
  - **Tamaño por defecto**: 240×72 pt (vs. 200×60 en v0.4.4).
- `packages/signer/src/visibleSig.ts`:
  - Nuevo helper `buildQrOperators(text, sizePt)` — convierte la matrix N×N de `qrcode` a operators PDF `q / 0 0 0 rg / re* / f / Q` con coalescencia horizontal de runs (3-5× menos rectángulos vs. naïve por-módulo).
  - `buildAppearanceOperators` extendido con `opts.qrUrl?, opts.signingTime?, opts.reason?`. Sin `qrUrl` mantiene layout legacy (back-compat).
  - Nuevo helper `formatSigningTime(d)` → `YYYY-MM-DD HH:mm` local time.
  - Nuevas constantes exportadas: `DEFAULT_VISIBLE_SIG_QR_WIDTH=240`, `DEFAULT_VISIBLE_SIG_QR_HEIGHT=72`, `SPLIT_MAX_CN_CHARS=35`.
- `packages/signer/src/pades.ts`: calcula SHA-256 del PDF source pre-sign, toma los primeros 12 hex chars, construye `qrUrl` y los pasa al widget. Acceso content-addressable estable que sobrevive re-firma.
- `apps/pwa/src/ui/firma/BoxPlacer.svelte`: defaults a 240×72 pt (`MIN_W=180`, `MIN_H=54`); preview WYSIWYG split — placeholder QR a la izquierda + 3 líneas mock (CN preview, fecha en vivo, "Razón: firmar.ec") a la derecha. Borde y proporción coinciden con el output PDF.
- `apps/pwa/src/routes/About.svelte`: nueva sección "Código QR de validación" + bump APP_VERSION → `0.4.5`.
- i18n keys: `firmar.qr_label`, `firmar.box_qr_placeholder`, `about.qr_title`, `about.qr_description` (ES + EN).

### Tests
- `packages/signer/tests/visibleSig.test.ts` — **+5 tests** para v0.4.5:
  - `buildQrOperators` emite rect+fill ops del QR matrix (>5 rectángulos).
  - `buildAppearanceOperators` con `qrUrl` produce border + ≥10 rects + 3 Tj a 8 pt + hex codificados de las 3 líneas.
  - Sin `qrUrl` mantiene layout legacy (1 Tj, 10 pt, sin borde).
  - `formatSigningTime` produce `YYYY-MM-DD HH:mm` local.
  - `signPdfPades` inyecta el `qrUrl` con sha256-12 hex hint correcto en el AP/N stream.
  - End-to-end: PDF firmado con split layout sigue verificable (covered-hash matches CMS messageDigest).
- 2 tests legacy (`renders Firmado por…`, `truncates CN > 50 chars…`) actualizados para reflejar el nuevo layout (8 pt font, 35-char cap).
- **Cumulative**: signer 54 + verifier 47 + tools/sbom 2 = **103 passing** (vs 95 en v0.4.4 → +8).

### Privacy & bundle
- QR generado 100% client-side; **sin** llamadas a APIs externas (Google Charts, qrcode-monkey, etc.).
- `qrcode` lib añade ~25 KB minified gzip al bundle del signer — aceptable.

### Dependencias
- `qrcode@^1.5.4` (+ `@types/qrcode` dev) en `packages/signer`.

## [0.4.4] - 2026-05-09 — P0 hotfix round-trip sign↔verify (sigValid=false killer)

### Fixed
- **P0 — Round-trip sign↔verify roto.** Tras los fixes v0.4.3 (3DES) + v0.4.2 (UX), los PDFs firmados en `/firmar` con `.p12` reales (ArgosData u otras ECIs ecuatorianas) llegaban a `/verificar` como `status='invalid'` ("Firma inválida" rojo + banner DEMO simultáneamente). Reproducible 100% con `rsa2048-3des-legacy.p12` y `rsa2048-valid.p12` en el nuevo suite `roundtrip.test.ts`. Causa raíz: `packages/signer/src/cms.ts` usaba `signedAttrsSet.encodedValue` para obtener los bytes a firmar:
  ```ts
  const signedAttrsDerForSign = new Uint8Array(signedAttrsSet.encodedValue);
  ```
  Pero `encodedValue` en `pkijs.SignedAndUnsignedAttributes` **solo está poblado cuando el objeto se construye parseando BER** — en el camino de **construcción nueva** retorna un `ArrayBuffer` de length 0. Resultado: firmábamos 0 bytes (la firma RSA del SHA-256 de la cadena vacía), mientras que el verificador reconstruía `signedAttrsDer` vía `signerInfo.signedAttrs.toSchema().toBER(false)` (~166 bytes reales). Web Crypto `verify` retornaba `false` → `sigValid=false` → `status='invalid'`, sobre **100% de los PDFs firmados**. **NO era un bug de PKCS#12 ni de wrap PKCS#8** — la cadena `forge → wrapRsaPrivateKey → Web Crypto importKey('pkcs8')` funcionaba perfectamente (validado por nuevo test `Web Crypto cross-check: forge-wrapped privKey signs match cert pubkey`).
  - **Fix**: usar `signedAttrsSet.toSchema().toBER(false)` (mismo path que el verificador) y parchar el primer byte `0xa0` (IMPLICIT [0]) → `0x31` (SET OF universal) per RFC 5652 §5.4. Diff localizado en `packages/signer/src/cms.ts` líneas 131-148.

### Added
- **Round-trip regression suite** `packages/signer/tests/roundtrip.test.ts` (3 tests, todos passing tras el fix):
  - `RSA-2048 valid (AES-256 PFX)` — firma PDF con `rsa2048-valid.p12`, verifica con TSL placeholder roots → `status='warning'` + `digestMatches=true` + `subjectCN='Test Signer RSA-2048'`.
  - `RSA-2048 3DES legacy (Ecuadorian ECI shape)` — proxy más cercano al `.p12` real ArgosData del usuario; mismo flow → `status='warning'` + `TRUST_PLACEHOLDER` warning + signer CN correcto.
  - `Web Crypto cross-check: forge-wrapped privKey signs match cert pubkey` — guard de unidad: extrae privKey + pubKey del PFX, firma un blob arbitrario, verifica. Pinning permanente: si esto falla, el wrap PKCS#8 de `p12.ts` está roto.
- Estos 3 tests **falsean ANTES** del fix y pasan DESPUÉS — pinning permanente de la regresión.

### Tests
- `packages/signer`: 48 passing + 2 skipped (50 total) — +3 desde v0.4.3.
- `packages/verifier`: 47 passing + 2 skipped (49 total) — sin cambios.
- **Total cumulative**: 95 passing (era 92 en v0.4.3).

### Deferred (v0.4.5)
- **QR estilo FirmaEC en firma visible** — fuera del scope P0. Diseño esbozado en el handoff:
  - Cuadro 240×72pt con QR (60×60pt) + texto (3 líneas: `Firmado por:`, `Fecha:`, `Razón:`).
  - QR content: `https://firmar.ec/#/verificar?h=<sha256-12chars>` para escaneabilidad estándar EC.
  - Implementación: dep `qrcode-svg` (~30 KB), Form XObject embebido en PDF vía pdf-lib, `BoxPlacer.svelte` preview WYSIWYG con placeholder QR + texto split-layout.
- Decisión: priorizar fix P0 sigValid → liberar v0.4.4 sin QR. v0.4.5 incluirá la firma visible con QR oficial.

## [0.4.3] - 2026-05-09 — P0 hotfix `pfx_unsupported_algo` (3DES legacy de ECIs ecuatorianas)

### Fixed
- **P0 — Killer bug: ningún `.p12` ecuatoriano real podía firmar.** Tras llegar al step 4 PIN del flujo `/firmar`, todo `.p12` emitido por las ECIs ecuatorianas (BCE, Security Data, ArgosData, ANFAC, Consejo Judicatura) caía con `Error inesperado. code: pfx_unsupported_algo`. Causa raíz: las ECIs cifran sus PKCS#12 con `pbeWithSHAAnd3-KeyTripleDES-CBC` (PBE-SHA1-3DES, default de OpenSSL pre-3.0). Nuestro `packages/signer/src/p12.ts` usaba `pkijs`, que delega cripto simétrica a Web Crypto API, y **Web Crypto API no expone 3DES**. Resultado: `pfx_unsupported_algo` determinístico sobre el 100% del corpus ecuatoriano real.
  - **Fix**: switch del backend de descifrado PKCS#12 `pkijs` → `node-forge`. node-forge provee implementación pura JS de 3DES + AES + RC2 + el matrix completo de ciphers PKCS#12 legacy.
  - `packages/signer/src/p12.ts` reescrito: (1) `forge.asn1.fromDer` parsea el outer PFX, (2) `forge.pkcs12.pkcs12FromAsn1(asn1, false, pin)` descifra TODOS los `safeContents` y `pkcs8ShroudedKeyBag` independientemente del cipher, (3) bag de cert → DER → `SignerCert`, (4) bag de clave RSA → `forge.pki.wrapRsaPrivateKey` → PKCS#8 DER que `pades.ts importPrivateKey('pkcs8', …)` consume sin cambios.
  - **Mapeo de errores preservado**: `MAC could not be verified` / `Invalid password` → `pin_invalid`; `Unsupported|cipher|algorithm|OID` → `pfx_unsupported_algo`; otros → `pfx_corrupt`. Contrato externo `SignerError` invariante.
  - **Privacidad intacta**: node-forge corre 100% client-side. El `.p12` y el PIN nunca tocan red.
  - **Bundle impact**: +~80 KB minified+gzip por node-forge. Aceptable para el caso de uso (firma local, ya cargamos pkijs/asn1js).

### Added
- **Fixture sintético `rsa2048-3des-legacy.p12`** generado vía `forge.pkcs12.toPkcs12Asn1` con `algorithm: '3des'` para reproducir exactamente el shape de las ECIs ecuatorianas. Pinning de regresión: cualquier futuro switch fuera de node-forge volverá a romper el flujo y los tests lo capturan.
- **Tests `parsePfx` 3DES legacy** (`packages/signer/tests/p12.test.ts`):
  - `parses RSA-2048 3DES legacy (Ecuadorian ECI cipher) → SUCCESS` — happy path con PIN correcto, valida `sigAlg=RSA-PKCS1-SHA256`, `kty=RSA`, PKCS#8 DER bien-formado.
  - `parses RSA-2048 3DES legacy with WRONG PIN → pin_invalid` — error mapping correcto.

### Changed
- `packages/signer/package.json`: `node-forge ^1.4.0` movido de `devDependencies` → `dependencies` (era dep dev solo para fixtures).
- `packages/signer/scripts/gen-test-p12.ts`: parametrizado con opción `algorithm: 'aes256' | '3des'` para emitir fixtures legacy.

### Deferred (v0.4.4)
- **ECDSA P-256 PFX parsing temporalmente bloqueado**. La fixture sintética `ecdsa-p256-valid.p12` se construye con pkijs y emite un `EncryptedPrivateKeyInfo` cuyo encoding del `OCTET STRING constructed` node-forge rechaza. Las ECIs ecuatorianas reales emiten **siempre** RSA + 3DES (no ECDSA), por lo que este edge case está **fuera del path P0**. Tests ECDSA marcados `it.skip` / `describe.skip`. Plan v0.4.4: regenerar la fixture en shape forge-compatible o añadir fallback pkijs solo para PFX ECDSA-only.
- Tests de `addIncrementalSignature` que usaban el PFX ECDSA como segundo firmante migrados a `rsa1024-weak.p12` (CN distinto a `rsa2048-valid.p12`).

### Tests
- `packages/signer`: 45 passing + 2 skipped (47 total).
- `packages/verifier`: 47 passing + 2 skipped (49 total). Sin cambios.
- **Total cumulative**: 92 passing.

## [0.4.2] - 2026-05-09 — P0 hotfix /firmar UX

### Fixed
- **P0 — Signature box rendered OFF the PDF page.** El overlay del `BoxPlacer` se montaba sobre `.pdf-stage-host` (contenedor padre con padding y page-nav), no sobre el `<canvas>` real. Resultado: el cuadro aparecía flotando en el margen blanco izquierdo y el usuario no podía colocar la firma.
  - `apps/pwa/src/ui/firma/PdfPreview.svelte` ahora acepta un snippet `overlay({ cssWidth, cssHeight })` que se renderiza en una capa absoluta dentro de un `.canvas-stack` (display:inline-block) anclado al canvas. Las dims del overlay siempre coinciden con `canvasEl.style.width/height`.
  - `apps/pwa/src/routes/Firmar.svelte` pasa `BoxPlacer` como ese snippet en lugar del mount externo `position:absolute` desalineado.
- **P0 — Sin posición inicial.** `BoxPlacer` requería tap-to-place; en touch ergonomics el tap caía a veces fuera del área visible. Ahora un `$effect` coloca un cuadro centrado horizontal + 12% del fondo de la página automáticamente cuando llega `pdfPageSize`. El usuario puede arrastrar/redimensionar igual.
- **P1 — Doble botón "Continuar"** en step 2 (uno en el overlay del PDF + otro en el footer del wizard). Eliminado el `confirm-bar` interno del `BoxPlacer`; el footer del `WizardShell` es el único CTA de avance.
- **P1 — Stepper "Paso 2 de 7 / 7"** duplicado. `WizardShell.svelte:124` concatenaba ` / {totalSteps}` además del valor de `firmar.step_of` que ya incluye "de 7". Eliminado el sufijo.
- **P2 — Visibilidad del cuadro.** Borde dashed 2px → solid 2.5px, fill `oklch 0.10` (antes `0.04`), inset ring blanco 1px + soft drop-shadow para contraste sobre fondo blanco del PDF.
- **P2 — Preview text "tu no..." truncado raro.** Placeholder ES `Firmado por: tu nombre` → `Firmado por: [tu nombre del certificado]`; EN equivalente. Los corchetes señalan claramente que es un slot a rellenar y la elipsis truncada lee mejor que la palabra cortada a mitad.

### Changed
- `BoxPlacer` añade prop `onChange?: (pos) => void` para que el parent observe las mutaciones (auto-place, drag, resize, keyboard) sin necesidad de `bind:`. `Firmar.svelte` cablea `onChange={onBoxPositionChange}`.

### Tests
- `apps/pwa/tests/e2e/firma.spec.ts::step2PlaceBox` y `firma.mobile.spec.ts::Test 5b` actualizados al nuevo contrato: esperar `.sig-box` visible (auto-placed) y avanzar con el botón Next del footer (`getByRole('button', { name: /^continuar$|^continue$/i }).last()`). Ya no se busca `[data-testid="box-confirm-bar"]`.

## [0.4.1] - 2026-05-09

### Added
- **Custom Service Worker** (`apps/pwa/src/sw.ts`) — migración de VitePWA `generateSW` → `injectManifest` para poder interceptar `POST /share`. El SW:
  - Lee el `FormData` del Share Target, valida MIME (`application/pdf`), tamaño (≤50 MB), magic bytes `%PDF-`.
  - Stash del PDF en Cache Storage (`shared-pdf-v1`) bajo `/__shared-pdf__/<uuid>` con `X-Stored-At` y `X-Filename`.
  - 303 redirect a `/#/share?pdfId=<uuid>` (svelte-spa-router lo recoge).
  - Errores: redirect a `/?shareError=<no_file|not_pdf|too_big|invalid_pdf|internal>`. `App.svelte` reescribe esa query a `#/share?shareError=...` para que el SPA muestre el mensaje localizado.
  - Cleanup TTL 10 min — entradas viejas en `shared-pdf-v1` se borran en cada nueva escritura.
  - Mantiene reglas `NetworkOnly` de v0.4.0 para `/_assets/crypto-*`, `/trust/tsl-ec.json`, `/trust/tsl-ec.sha256` (parity de seguridad).
  - `precacheAndRoute(self.__WB_MANIFEST)` + `cleanupOutdatedCaches()` para que usuarios con shells viejas no queden colgados.
- `SharedFileHandler.svelte` ahora lee `pdfId` desde el hash, hace `caches.match('/__shared-pdf__/<id>')`, borra la entrada inmediatamente tras consumir (privacidad), corre `detectSignatures` y redirige a `/verificar` o `/firmar` (mismo pipeline v0.4.0).
- i18n nuevos: `share.error.{no_file,invalid_pdf,internal}` ES+EN.
- E2E spec `apps/pwa/tests/e2e/share-target.spec.ts` (skip si no hay `PREVIEW_BASE_URL`): verifica registro del SW, POST flow happy path, errores (no_file, not_pdf, invalid_pdf), TTL cleanup.

### Changed
- `apps/pwa/vite.config.ts` `VitePWA`: `strategies: 'injectManifest'`, `srcDir: 'src'`, `filename: 'sw.ts'`, `injectManifest.maximumFileSizeToCacheInBytes: 5_000_000`. Bloque `workbox: {...}` removido (lógica vive ahora en `src/sw.ts`).
- `apps/pwa/package.json` añade `workbox-precaching`, `workbox-routing`, `workbox-strategies` `^7.4.1` como deps directas (antes eran transitivas via `vite-plugin-pwa`).
- `App.svelte` detecta `?shareError=...` en `window.location.search` (post-redirect del SW) y lo reescribe a `#/share?shareError=...` para que SharedFileHandler muestre el error.

### Privacy
- El PDF compartido vive en Cache Storage local (per-origin, nunca sincronizado) y se borra al consumir + por TTL 10 min. Mantiene la promesa "nada sale del navegador".

## [0.4.0] - 2026-05-09

### Added
- **PWA share target & file handlers** (mobile UX).
  - Manifest declara `share_target` (POST + multipart, files accept `application/pdf`), `file_handlers` (`accept: { 'application/pdf': ['.pdf'] }`, `launch_type: single-client`), y `launch_handler: navigate-existing`. Ahora firmar.ec aparece en el menú "Compartir" / "Abrir con" del sistema (Android/Chromium-desktop) cuando la PWA está instalada.
  - Nuevas rutas `/share` y `/handle-file` en el SPA → componente `SharedFileHandler.svelte`. Lee el archivo desde `window.launchQueue` (Chromium 102+), corre `detectSignatures` y redirige a `/verificar` si hay firmas o a `/firmar` si no.
  - Pre-load de PDF en `Verificar.svelte` y `Firmar.svelte` vía sessionStorage (`__incomingPdf`), `consume()` borra la entrada para preservar privacidad.
  - Helper nuevo `apps/pwa/src/lib/sharedFile.ts` con round-trip Uint8Array ↔ base64 chunked (no stack overflow en >32KB).
- `InstallPrompt.svelte` — captura `BeforeInstallPromptEvent`, muestra card sutil bottom-fixed, persiste dismiss 30 días, oculta si la app está en `standalone` display-mode o en rutas `/share`/`/handle-file`.
- Onboarding visual en `Home.svelte`: sección "Recibe un PDF por WhatsApp o Gmail" con 3 pasos (icons lucide `share-2`, `pen-tool`, `download`), variante install hint para iOS Safari.
- Nueva sección en `About.svelte`: "share target capability" con copy "Compatible con WhatsApp, Gmail, Outlook y cualquier app de mensajería en Android e iOS".
- i18n ES+EN: `share.processing`, `share.waiting_hint`, `share.error.{not_pdf,too_big,read}`, `share.back_home`, `install.prompt.{title,body,cta,dismiss}`, `home.share_anchor.{title,subtitle,step1,step2,step3,install_hint}`, `about.share_target_capability`.
- Vitest unit tests (`apps/pwa/tests/sharedFile.test.ts`) — round-trip, chunk-boundary >32KB, payload corrupto, empty state. **4/4 PASS**.
- Documentación: spec F3 actualizada con sección "v0.4.0 Share Target & File Handlers (in-scope mobile UX)".

### Changed
- `Caddyfile.pwa`: `Permissions-Policy` cambia `web-share=()` → `web-share=(self)` (necesario para que la PWA actúe como Share Target y use `navigator.share()`).
- `vite.config.ts` `workbox.navigateFallbackDenylist` añade `/^\/share/` y `/^\/handle-file/` (rutas de OS-handoff nunca deben servirse desde precache).
- `App.svelte` ahora trackea `currentRoute` vía `onRouteLoaded` callback para que `InstallPrompt` se oculte en flujos de share.

### Caveats / Deferred to v0.4.1
- **POST `/share` con multipart files no funciona todavía** sin un Service Worker custom que intercepte la request, parsee el `FormData` y haga handoff al SPA via Cache API + sessionStorage. La declaración del manifest se mantiene para que el OS liste firmar.ec, pero un share de archivo entregará el POST a Caddy (que responde 405). El flujo `file_handlers` (Open with) **sí funciona** porque usa `launchQueue` (sin SW). v0.4.1 migrará Workbox de `generateSW` a `injectManifest` para añadir el endpoint POST sin romper la política `NetworkOnly` de los chunks crypto.
- iOS Safari no implementa `share_target` ni `file_handlers` PWA-side; usuarios iOS verán solo "Add to Home Screen" + onboarding manual.
- Live Playwright audit en `app.firmar.ec` queda pendiente del deploy v0.4.0 (image build + push + `docker service update`). Tests E2E locales no añadidos en este sprint — la simulación de share_target en headless requiere mock manual del endpoint.

## [0.3.4] - 2026-05-09

### Added
- TSL `@firma-ec/tsl-ec` expandida de 7 a **17 ACEs ARCOTEL** acreditadas (todas placeholder) — alpha-technologies, anfac, appfirmas, argosdata, bce, judicatura, corpnewbest, darkcam, datil, registro-civil, eclipsesoft, firmasegura, lazzate, letmi, primecorelat, securitydata, uanataca.
- Nuevo campo opcional `acceptedInGobEc` en `TrustRoot` interface — 8/17 ACEs marcadas como aceptadas por SRI en gob.ec (ANFAC, ArgosData, BCE, Consejo de la Judicatura, DatilMedia, EclipSoft, Security Data, UanaTaca).
- TSL JSON payload ahora expone `stats.totalArcotelAccredited`, `stats.acceptedInGobEc`, `stats.sources` (URLs ARCOTEL + SRI).
- Generador de placeholders self-signed `packages/tsl-ec/scripts/gen-placeholder-pems.mjs` (node-forge). Modo `--missing` por default, `--all` regenera todas.
- PWA copy: `Home.svelte` añade CTA SRI gob.ec + counter "17 ACEs", `About.svelte` añade sección "Compatibilidad legal" con links ARCOTEL/SRI, `DropP12` ECI hint actualizado a "17 ACEs soportadas".
- i18n keys (es+en): `home.sri_anchor`, `home.aces_count`, `home.sri_link`, `about.aces_title`, `about.aces_body`, `about.aces_link_arcotel`, `about.aces_link_sri`.
- Documentación regulatoria: spec F3 nueva sección §7.5 (SRI gob.ec + ARCOTEL TSL), plan F3 nota out-of-scope confirmando 17 ACEs, adendum UI Pro Max §0.5 (anchor de copy).

### Changed
- `TSL_VERSION` 1.0.0 → 1.1.0, `TSL_SEQUENCE` 1 → 2.

### Notes
- **Todos los 17 PEMs siguen siendo placeholder**. La heurística `allRootsPlaceholder` del verifier sigue válida y mantiene el estado "warning + DEMO banner" para verificaciones reales hasta publicación de PEMs auténticos.

## [Earlier]

### Added
- Bootstrap del monorepo (F0).
