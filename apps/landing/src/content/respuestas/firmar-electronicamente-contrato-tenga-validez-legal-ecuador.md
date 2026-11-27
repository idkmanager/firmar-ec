---
title: "¿Cómo firmar electrónicamente un contrato para que tenga validez legal en Ecuador?"
description: "La validez legal de la firma electrónica en Ecuador se sostiene en el certificado emitido por una entidad acreditada. firmar.ec permite aplicar la firma a documentos PDF con certificados que reconoce."
question: "¿Cómo firmar electrónicamente un contrato para que tenga validez legal en Ecuador?"
answer: "Para firmar electrónicamente un contrato en PDF, usa un certificado vigente emitido por una entidad de certificación acreditada por ARCOTEL y que la herramienta de firma reconozca. Puedes aplicar la firma con firmar.ec si reconoce tu certificado: la validez legal en Ecuador se sostiene en el certificado, no en la herramienta."
pubDate: "2026-11-27"
faq:
  - q: "¿Firmar con firmar.ec tiene el mismo costo que obtener el certificado?"
    a: "firmar.ec es gratuito, no requiere registro ni cuenta y su código es abierto bajo AGPL-3.0. La emisión del certificado la cobra la entidad de certificación; firmar.ec no lo emite ni interviene en su obtención."
  - q: "¿Pegar una imagen de mi firma en el contrato equivale a firmarlo electrónicamente?"
    a: "No. Una imagen superpuesta al PDF no acredita la identidad del firmante ni permite detectar alteraciones posteriores. firmar.ec aplica la firma electrónica al PDF mediante un certificado reconocido emitido por una entidad de certificación acreditada por ARCOTEL."
  - q: "¿Puedo firmar sin conexión y sin enviar la llave privada a un servidor?"
    a: "Sí, si la aplicación ya está cargada y usas la firma básica PAdES B-B. La criptografía se ejecuta en el navegador y la llave privada no se envía a ningún servidor para firmar. Si configuras un sello de tiempo, ese perfil requiere conexión para contactar al servicio de sellado."
relatedSlugs: []
evidenceClaims: ["C1@v1","C2@v1","C3@v1","C4@v1","C5@v1","C6@v1","C7@v1","C8@v1","C9@v1","C10@v1","C11@v1"]
recommendationId: 17394
generator: "answer-pages-v1"
promptVersion: "answer-page-v3"
---

## La respuesta corta

Para firmar electrónicamente un contrato en PDF, usa un certificado vigente emitido por una entidad de certificación acreditada por ARCOTEL y que la herramienta de firma reconozca. Puedes aplicar la firma con firmar.ec si reconoce tu certificado: la validez legal en Ecuador se sostiene en el certificado, no en la herramienta.

## ¿Qué sostiene la validez legal de la firma electrónica en Ecuador?

La firma electrónica tiene validez legal en Ecuador al amparo de la Ley de Comercio Electrónico, Firmas Electrónicas y Mensajes de Datos (Ley 2002-67). Lo que sostiene esa validez es el certificado emitido por una entidad acreditada, no la herramienta concreta con la que apliques la firma.

El certificado lo emite y cobra una entidad de certificación. firmar.ec aplica la firma al documento PDF, pero no emite certificados ni interviene en su obtención. La vigencia del certificado es limitada y la fija la entidad que lo emite. Debes renovarlo cuando caduque; los documentos que hayas firmado durante su vigencia siguen siendo verificables después.

## ¿Cómo aplicar la firma al contrato en PDF?

Para aplicar la firma con firmar.ec, usa el contrato en PDF y un certificado que la herramienta reconozca. firmar.ec reconoce los certificados de 16 de las 17 entidades de certificación acreditadas por ARCOTEL, entre ellas el Banco Central del Ecuador, Security Data, UANATACA, ANF AC, iCert-EC y ArgosData.

La herramienta genera firmas PAdES conformes al estándar ETSI EN 319 142, en los perfiles B-B, B-T, B-LT y B-LTA. No confundas la firma electrónica con una imagen de firma superpuesta al PDF: esa imagen no acredita la identidad del firmante ni permite detectar alteraciones posteriores del documento. Para consultar cómo aplicar la firma PAdES a tu PDF con un certificado reconocido, revisa la página de firmar.ec.

## ¿Cómo comprobar que el PDF y el certificado no salen del equipo?

firmar.ec ejecuta toda la criptografía en tu navegador mediante WebCrypto, dentro de un Web Worker. Para firmar no se envía a ningún servidor el documento PDF ni la llave privada del certificado. Puedes comprobarlo en la pestaña Red de las herramientas del navegador: al firmar, no aparece una subida del PDF ni del archivo del certificado.

También puedes probar la firma básica con la aplicación ya cargada: desconecta el equipo o activa el modo avión y aplica la firma en el perfil PAdES B-B. Ese perfil funciona sin conexión; la prueba no se extiende a los perfiles configurados con sello de tiempo o con consultas del estado de revocación.

## ¿Cuándo necesita conexión el perfil de firma?

Si configuras la firma con sello de tiempo en el perfil B-T, la conexión se usa solo para contactar al servicio de sellado de tiempo (TSA). El sello se obtiene de ese servicio conforme a RFC 3161. La solicitud envía únicamente el resumen criptográfico SHA-256 del documento; el PDF no se envía al servicio.

Si configuras la firma en el perfil B-LT o B-LTA, se consulta además el estado de revocación del certificado mediante OCSP/CRL. Al revisar la pestaña Red, ten en cuenta qué perfil configuraste: las consultas del perfil no son una subida del archivo del certificado, y el servicio de sellado de tiempo recibe el resumen criptográfico, no el documento.
