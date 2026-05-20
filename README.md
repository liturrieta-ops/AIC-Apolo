ADR-006: Implementación de un MCP Gateway Centralizado con Auditoría y Control de Cuotas
Estatus: Aceptado

Contexto Asociado: Extensión del modelo de seguridad basado Validación de identidad, Validación JWT 

1. Contexto (Context)
El sistema actual delega en un Agente LLM el uso de herramientas a través del protocolo MCP (Model Context Protocol) para consultar bases de datos financieras.
El análisis de amenazas previos (Threat Modeling) arrojó múltiples riesgos calificados como "Incierto" o "Parcial" que comprometen la integridad del ecosistema:

- Falta de Auditoría de Acciones: No existe un registro centralizado, inalterable y homogeneizado que mapee con precisión qué usuario aprobó manualmente una acción y qué herramienta MCP exacta se ejecutó.
- Agotamiento de Recursos (Token Exhaustion): El sistema es vulnerable a ataques de denegación de servicio económico o técnico debido a la falta de cuotas de consumo por usuario a nivel de llamadas de agentes.
- Principio de Menor Privilegio Débil: Las herramientas (plugins) MCP corren el riesgo de interactuar de forma promiscua si no se introduce una capa de control perimetral que restrinja qué herramientas puede ver
  o invocar el Agente según el rol del cliente autenticado.
- Dejar la responsabilidad de la auditoría y el control de consumo del lado del cliente (Frontend) o directamente en los plugins individuales de la base de datos incrementa el acoplamiento y fragmenta la seguridad.

2. Decisión (Decision)
Se propone la creación e implementación de un MCP Gateway Centralizado como un microservicio independiente que actuará como intermediario obligatorio (Proxy) entre el Agente LLM y los servidores que exponen
las herramientas MCP.

Este componente ejecutará de manera estricta las siguientes políticas:
- Interceptación y Log de Auditoría Obligatorio: Cada solicitud de ejecución de herramienta será registrada en un log centralizado e inmutable de auditoría. 
  El registro incluirá: Timestamp, Customer_id, herramienta invocada, parámetros de entrada y el hash de aprobación que valide que pasó por el control Human-in-the-Loop.
- Control de Cuotas y Limitación por Usuario (Rate Limiting Agentic): Se implementará un mecanismo de almacenamiento en caché (ej. Redis) en el Gateway para contabilizar y
  limitar el número de llamadas a herramientas y el consumo estimado de tokens por identificador de cliente de forma diaria/por hora.
- Aislamiento de Plugins y Control de Acceso (RBAC para Tools): El Gateway funcionará como un Allowlist dinámico. Evaluará los claims del token JWT del usuario y solo expondrá u otorgará permisos de ejecución
  sobre herramientas específicas que correspondan a su nivel de acceso, mitigando el riesgo de Excessive Agency.

3. Consecuencias 
Positivas (+)
- Resolución de Brechas del Threat Model: Cambia el estado de "Auditoría de acciones", "Token exhaustion" y "Aislamiento de plugins" de Incierto/Parcial a Mitigado.
- Defensa en Profundidad: Si un atacante logra saltarse el Human-in-the-Loop mediante una inyección de prompts muy avanzada, el Gateway actuará como una última línea de
  defensa física bloqueando la petición si rompe las reglas de cuota o accesos del rol.
- Trazabilidad Forense: Permite generar análisis precisos de uso y auditorías de cumplimiento normativo ante reguladores financieros.

Negativas (-)
- Punto Único de Falla (Single Point of Failure): Si el MCP Gateway se cae, el Agente conversacional pierde la capacidad de usar cualquier herramienta interna. 
(Mitigación: Requiere despliegue con alta disponibilidad y redundancia).
 - Incremento de Latencia: Al introducir un salto de red adicional entre el Agente, el Gateway y el servidor MCP definitivo, se añadirá un pequeño margen de milisegundos a la respuesta del bot.
