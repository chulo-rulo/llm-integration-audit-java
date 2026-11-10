# llm-integration-audit-java

[English](README.md)

Skill de Claude que audita de extremo a extremo cómo una aplicación Java se integra con modelos de lenguaje: cómo entra la entrada, cómo se construyen los prompts, cómo se llama al modelo, cómo se comportan las herramientas, la memoria y la recuperación (RAG), cómo se trata y entrega la salida, y cómo falla, cuánto cuesta y cómo maneja los datos personales todo el flujo. También cubre aplicaciones que exponen o consumen MCP y montajes multiagente.

Versión 1.0.0 · Apache-2.0 · Copyright (c) 2026 Igor Denyshchyk · Denis Art (DsA)

## Qué cubre

- Clientes: Spring AI, LangChain4j (directo, Quarkus, Micronaut, Helidon), Semantic Kernel, Embabel, LangGraph4j, Google ADK, Koog, MCP Java SDK, SDKs de Anthropic, OpenAI, Google Gen AI, AWS Bedrock y Azure, modelos locales y llamadas HTTP directas a proveedores.
- 20 bloques de revisión (B00–B19): secretos, acceso y consumo, construcción de prompts, RAG, herramientas y agencia, tratamiento de la salida, memoria y cachés, resiliencia, coste y observabilidad, datos personales, canal cliente, tests, dependencias, guardrails, multimodal, ciclo de vida del modelo, tareas en segundo plano, MCP.
- Mapeado a OWASP Top 10 for LLM Applications (2025 y 2026) y OWASP Top 10 for Agentic Applications.
- Dos modos: Quick (primera pasada o pull request) y Full.

## Cómo funciona

1. Hace una única ronda de preguntas de contexto: quién accede a las funciones con LLM, qué datos circulan, qué puede provocar el LLM, multi-tenant, madurez del proyecto, modo.
2. Detecta el stack y las versiones.
3. Localiza cada punto donde se llama a un modelo, dibuja el flujo con sus fronteras de confianza y construye una matriz de cobertura y (en modo Full) un inventario de todos los almacenes con datos de usuario, incluidas las tablas que crean los frameworks en tiempo de ejecución.
4. Recorre cada flujo con los bloques que aplican.
5. Verifica cada hallazgo, busca mitigaciones en otras partes y comprueba el comportamiento del framework contra la versión de la librería usada.
6. Redacta un informe con evidencias (`archivo:línea` y traza), severidad, confianza (Confirmado / Probable / Requiere comprobación en ejecución), cadenas de ataque y comprobaciones pendientes. Pregunta antes de guardarlo como `LLM_AUDIT.md`.

La auditoría es de solo lectura: nunca modifica el proyecto y solo ejecuta código con permiso explícito.

## Instalación

- Apps de Claude (claude.ai, escritorio): comprime la carpeta `llm-integration-audit-java` en zip y súbela en Personalizar > Skills. Si hay instalada una versión antigua de un solo archivo, sustitúyela.
- Claude Code: copia la carpeta a `~/.claude/skills/` (personal) o a `.claude/skills/` dentro de un repositorio (proyecto).

## Uso

Pide a Claude algo como «audita la integración con LLM de este proyecto» o «revisa cómo llama esta app al modelo», con el proyecto accesible. Responde a las preguntas de contexto: de ellas dependen las severidades.

## Contenido

```
SKILL.md                     flujo, reglas, severidad, quick checks, bloques
references/detection.md      patrones de detección por cliente, dónde viven los controles
references/report-template.md estructura del informe, incluido el caso sin hallazgos
references/owasp-map.md      mapeo OWASP LLM 2025/2026 y Agentic
references/probes.md         ideas de pruebas adversarias
CHANGELOG.md                 versiones y notas de mantenimiento
```

## Validación hasta ahora

Medida durante el desarrollo, con sus límites:

- Dos repositorios de prueba con fallos sembrados (Spring AI, LangChain4j, servidor MCP; 63 fallos y 27 señuelos en total): 41,5 de 42 y 21 de 21 fallos encontrados, sin falsos positivos en los señuelos (v0.2, v0.3 y de nuevo con la v0.5.1).
- Un proyecto multi-tenant casi limpio (Spring AI): encontró los dos fallos sembrados, no reportó ningún señuelo ni inventó hallazgos; los hallazgos adicionales eran defectos reales, verificados contra las fuentes del framework. La severidad salió un nivel alta en los hallazgos de consumo, lo que dio lugar a las reglas de calibración de la 0.5.2.
- Modo Quick sobre el primer repositorio de prueba: 42 de 42 fallos encontrados, sin falsos positivos en los señuelos.
- Tres proyectos públicos de código abierto: unos 25 hallazgos revisados a mano como reales, ninguno falso.
- Un proyecto privado en producción: 11 de 13 problemas conocidos con la primera versión; una vez corregidos, la re-auditoría con la 0.5.0 encontró 3 problemas medios más (límites de llamadas a herramientas configurados que no llegaban al cliente usado, memoria duplicada en el respaldo entre proveedores, texto personal etiquetado como impersonal) que la primera versión no podía detectar.

Límites: los repositorios de prueba los escribió el mismo autor que la skill; no se ha probado el informe de un proyecto sin hallazgos; solo Spring AI y LangChain4j se han probado a fondo. La mejor prueba es tu propio proyecto: se agradecen, como issues, resultados con otros clientes, fallos no detectados, falsos positivos y desacuerdos de severidad.

## Limitaciones

- Es una guía de revisión, no una garantía de seguridad. Los hallazgos salen de lectura estática. Un informe limpio no significa que la integración sea segura.
- El comportamiento del modelo, la configuración del proveedor y la infraestructura delante de la app no se pueden verificar desde el código; el informe los lista como comprobaciones en ejecución.
- Los puntos legales y regulatorios (RGPD, AI Act de la UE, términos del proveedor) se reportan como elementos a verificar, nunca como asesoramiento legal.
- Los valores por defecto de los frameworks y los estándares cambian. La skill agrupa los datos con fecha en una sola sección; revisa la fecha «Facts last verified» en `SKILL.md`.

## Licencia

Apache License 2.0. Ver `LICENSE` y `NOTICE`.

## Autor

Igor Denyshchyk · Denis Art (DsA)
