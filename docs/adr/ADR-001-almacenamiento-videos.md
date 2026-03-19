ADR-001: Estrategia de almacenamiento de videos
Aceptado

📅 Fecha
2026-03-17


Contexto :

La plataforma EducaMás necesita definir una estrategia para almacenar y distribuir videos educativos.
Factores relevantes:
🎥 Volumen estimado: 500 horas de video/año
👥 Usuarios concurrentes: 10.000 en horas peak
💰 Presupuesto: limitado
⚡ Rendimiento: baja latencia requerida
🔐 Seguridad: evitar descargas no autorizadas
👨‍💻 Equipo: experiencia en servidores, poca en CDN


Alternativas:

Opción A: Nube + CDN

Uso de:
Amazon S3
Amazon CloudFront

Pros:
Escalabilidad automática
Alta disponibilidad
Baja latencia global
Sin mantenimiento de infraestructura

Contras:
Dependencia del proveedor
Costos variables




Opción B: Servidores propios

Pros:
Control total
Costos iniciales definidos

Contras:
Baja escalabilidad
Alto costo inicial
Riesgo de fallas
Mantenimiento constante





Decisión:
Se selecciona la Opción A: almacenamiento en la nube con CDN.

Justificación:

Esta opción permite:
Escalar automáticamente ante alta demanda
Reducir costos operativos
Mejorar la experiencia del usuario
Evitar gestión de infraestructura

Consecuencias
Positivas:
Alta escalabilidad
Alta disponibilidad
Mejor rendimiento

Negativas
Dependencia de AWS
Costos variables a largo plazo

Impacto técnico
Integración con APIs cloud
Cambio hacia arquitectura cloud-native

Riesgos
Vendor lock-in
Mala configuración de costos
Resultado esperado
Plataforma escalable
Streaming eficiente

Base para crecimiento futuro
