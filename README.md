![image](https://github.com/user-attachments/assets/0e1b7f56-5d25-435c-b69d-71579fc23b24)


# 1. RESUMEN EJECUTIVO

## 1.1 Propósito de la auditoría
El propósito de esta auditoría fue evaluar la seguridad, la eficiencia operativa y el cumplimiento de buenas prácticas en el proceso de despliegue continuo automatizado de entornos WordPress mediante la solución **Chef_Vagrant_Wp** implementada por DevIA360. Se revisó el código fuente, los scripts de aprovisionamiento, las configuraciones declarativas y la evidencia técnica al replicar el entorno en un laboratorio de pruebas.

---

## 1.2 Alcance técnico resumido
- Despliegue de tres máquinas virtuales (**wordpress**, **database** y **proxy**) en la red `192.168.56.0/24` usando `vagrant up`.
- Análisis de configuraciones dentro del `Vagrantfile`, el archivo `.env` y los data bags de Chef para identificar parámetros inseguros.
- Ejecución de pruebas funcionales, de integración y de infraestructura mediante el script `tests.sh` y Serverspec.

---

## 1.3 Principales hallazgos
1. Presencia de credenciales sensibles en texto plano dentro de archivos Chef (data bags, `.env`) — **riesgo alto (25)**  
2. Puertos abiertos sin restricciones y ausencia de firewall en la configuración de Vagrant — **riesgo alto (20)**  
3. Falta de registros de auditoría persistentes durante el aprovisionamiento — **riesgo alto (16)**  
4. Uso de versiones de Apache, MySQL y Ruby desactualizadas sin ciclo de parches — **riesgo alto (20)**  
5. Entorno único sin segmentación clara entre desarrollo, pruebas y producción — **riesgo alto (20)**  
6. Cobertura de pruebas limitada, sin validaciones negativas ni de seguridad — **riesgo medio (12)**

---

## 1.4 Indicadores clave de desempeño (KPI)
- Se identificaron **5 riesgos críticos** (nivel alto ≥ 20) y **1 riesgo medio**, según la matriz OWASP Risk Rating.  
- No se generaron costos adicionales en licencias gracias al uso de componentes open source.  
- El **53% de las organizaciones** con pipelines CI/CD sin controles de seguridad han reportado incidentes (State of DevOps 2023).  
- Más del **90% de las pruebas funcionales** se superaron, aunque menos del 10% cubrieron escenarios de fallo o de seguridad (según `tests.sh`).

---
# 2. ANTECEDENTES

## 2.1 Contexto general de la entidad
DevIA360 es una organización peruana ubicada en Lima que se dedica al desarrollo de soluciones con inteligencia artificial y a brindar servicios de transformación digital. Su portafolio abarca proyectos de presencia web, análisis de datos y automatización de procesos para clientes medianos tanto a nivel nacional como internacional.

---

## 2.2 Naturaleza de sus sistemas de información
El sistema clave evaluado en esta auditoría es **Chef_Vagrant_Wp**, un conjunto de scripts y recetas Chef que, integrados con Vagrant, posibilitan el aprovisionamiento automatizado de un entorno WordPress conformado por un servidor web, base de datos y proxy inverso. Este sistema está incorporado dentro del pipeline de integración y entrega continua (CI/CD) de la empresa, y sirve para desplegar tanto sitios de demostración interna como entornos de pruebas previas a producción.

---

## 2.3 Estructura organizativa relevante
- **Dirección General**  
- **Departamento de Tecnología e Innovación** (liderado por el CTO), que integra:  
  - **Equipo de Desarrollo**  
  - **Equipo DevOps**, encargado de los pipelines de integración y despliegue continuo, así como del mantenimiento de Vagrant y Chef  
  - **Equipo de Seguridad de la Información**, responsable de definir políticas, revisar configuraciones y coordinar la respuesta ante incidentes

---

## 2.4 Antecedentes de auditorías previas
Hasta el momento, DevIA360 no había sido objeto de auditorías externas formales en sus procesos DevOps. Solo se habían realizado revisiones internas parciales de código y de buenas prácticas. Esta evaluación representa la primera auditoría integral de seguridad y cumplimiento aplicada al entorno **Chef_Vagrant_Wp**.

---
# 3. OBJETIVOS DE LA AUDITORÍA

## 3.1 Objetivo general
Realizar una evaluación integral de los procesos, controles y configuraciones implementados en el entorno de despliegue continuo **Chef_Vagrant_Wp** de DevIA360, con el propósito de determinar su nivel de seguridad, eficiencia operativa y conformidad con las mejores prácticas y normativas aplicables.

---

## 3.2 Objetivos específicos
1. Comprobar la protección de la información, garantizando la confidencialidad, integridad y disponibilidad de los datos durante el aprovisionamiento y operación del entorno.  
2. Analizar los mecanismos de continuidad del negocio (copias de seguridad, recuperación ante desastres, redundancia) para asegurar la resiliencia del servicio WordPress.  
3. Revisar la gestión de cambios y configuración, verificando que las modificaciones en los scripts Chef, el `Vagrantfile` y la infraestructura estén correctamente aprobadas, versionadas y probadas.  
4. Corroborar el cumplimiento normativo y la alineación con marcos de referencia reconocidos (ISO 27001, ITIL 4, OWASP DevSecOps, NIST SP 800-53).  
5. Validar la integridad y disponibilidad de los datos alojados en la base de datos MySQL y servidos por Apache, mediante pruebas de consistencia y monitoreo de rendimiento.  
6. Identificar riesgos residuales y oportunidades de mejora para robustecer la postura de seguridad y la eficiencia operativa de DevIA360.

---
# 4. ALCANCE DE LA AUDITORÍA

## 4.1 Ámbitos evaluados
- **Tecnológico**: Incluye la infraestructura virtual desplegada con Vagrant, las recetas y cookbooks de Chef, la configuración de los servidores WordPress, la base de datos MySQL y el proxy inverso (Apache / Nginx).  
- **Organizacional**: Se evaluaron procesos y responsabilidades de los equipos de DevOps y Seguridad de la Información, flujos de trabajo de integración y entrega continua, así como las políticas internas de TI.  
- **Normativo**: Se verificó la conformidad con marcos y estándares relevantes (ISO 27001, ISO 22301, ITIL 4, NIST SP 800-53, OWASP DevSecOps).  
- **Operativo**: Se revisaron procedimientos de respaldo, gestión de incidentes, monitoreo y registro de actividades (logging) a lo largo del ciclo de vida del entorno.

---

## 4.2 Sistemas y procesos incluidos
- **Pipeline CI/CD Chef_Vagrant_Wp**: encargado del aprovisionamiento automático de las máquinas virtuales wordpress, database y proxy.  
- **Repositorio de código y control de versiones**: incluyendo ramas, revisiones de pull requests y gestión de cambios en cookbooks, `Vagrantfile` y scripts auxiliares.  
- **Mecanismos de backup y recuperación**: tareas relacionadas con copias de seguridad de la base de datos MySQL y de los volúmenes de la máquina web.  
- **Plataforma de monitoreo y registros**: herramientas utilizadas para vigilar la disponibilidad, el rendimiento y la generación de alertas de seguridad.

---

## 4.3 Unidades o áreas auditadas
- **Equipo DevOps**: responsable del mantenimiento del pipeline de CI/CD y de la infraestructura como código.  
- **Equipo de Seguridad de la Información**: encargado de definir políticas, revisar configuraciones y gestionar la respuesta ante incidentes.  
- **Departamento de Tecnología e Innovación**: supervisa la estrategia general de TI y coordina la ejecución de iniciativas tecnológicas.

---

## 4.4 Periodo auditado
La auditoría abarcó las actividades, configuraciones y evidencias generadas entre el **1 de marzo y el 27 de junio de 2025**, considerando la versión vigente de **Chef_Vagrant_Wp** y todas las operaciones de despliegue llevadas a cabo durante ese periodo.

---
# 5. NORMATIVA Y CRITERIOS DE EVALUACIÓN

## 5.1 Normas y marcos internacionales
- **COBIT 2019**: marco para el gobierno y la gestión de TI enfocado en la generación de valor.  
- **ISO/IEC 27001:2022**: establece requisitos para implementar, mantener y mejorar un Sistema de Gestión de Seguridad de la Información (SGSI).  
- **ISO/IEC 27002:2022**: guía de buenas prácticas para controles de seguridad de la información.  
- **ISO 22301:2019**: orientado a la gestión de la continuidad del negocio.  
- **NIST SP 800-53 Rev. 5**: define controles de seguridad y privacidad para sistemas de información federales.  
- **ITIL 4**: mejores prácticas en la gestión de servicios de TI.  
- **OWASP DevSecOps Maturity Model**: recomendaciones de seguridad para pipelines de integración y entrega continua.

---

## 5.2 Normativa nacional
- **Ley N.º 29733** — Ley de Protección de Datos Personales del Perú y su reglamento (D.S. 003-2013-JUS).  
- **Ley N.º 30424** — sobre responsabilidad administrativa de personas jurídicas y programas de cumplimiento.

---

## 5.3 Políticas y procedimientos internos de DevIA360
- Política de Seguridad de la Información, versión **2025-01**  
- Procedimiento de Gestión de Cambios TI, versión **2025-02**  
- Estándar de Desarrollo Seguro y DevOps, versión **2025-01**

---

## 5.4 Criterios de evaluación
- Clasificación y priorización de riesgos aplicando la metodología **OWASP Risk Rating**, considerando la tolerancia al riesgo establecida por el Comité de Seguridad de DevIA360.  
- Incorporación de buenas prácticas de Infraestructura como Código (IaC) recomendadas por HashiCorp y Chef Software.

---
# 6. METODOLOGÍA Y ENFOQUE

## 6.1 Enfoque adoptado
Se implementó un enfoque mixto, combinando la perspectiva basada en riesgos con la perspectiva de cumplimiento:  
- **Basado en riesgos**: para identificar, analizar y priorizar amenazas que pudieran impactar la confidencialidad, integridad y disponibilidad del entorno **Chef_Vagrant_Wp**.  
- **Basado en cumplimiento**: para comprobar la alineación con los requisitos definidos en los marcos y normas mencionados anteriormente (COBIT 2019, ISO/IEC 27001:2022, Ley 29733, entre otros).

---

## 6.2 Etapas de la auditoría
1. **Planificación**: definición del alcance, objetivos, recursos y cronograma (del 1 de marzo al 27 de junio de 2025).  
2. **Levantamiento de información**: recopilación de evidencias mediante entrevistas, revisión documental y acceso controlado a los sistemas.  
3. **Ejecución de pruebas técnicas**: realización de análisis de vulnerabilidades, revisión de configuraciones y evaluación de los controles.  
4. **Evaluación y correlación**: análisis de los hallazgos en relación con los criterios normativos y el apetito de riesgo definido por DevIA360.  
5. **Informe**: elaboración de la documentación con resultados, conclusiones y recomendaciones, plasmadas en este reporte.

---

## 6.3 Métodos aplicados
- Entrevistas a personal de TI, DevOps y Seguridad de la Información para comprender procesos y controles vigentes.  
- Ejecución de pruebas técnicas:  
  - Análisis de registros y correlación de eventos  
  - Escaneos de vulnerabilidades mediante InSpec, OpenVAS y nmap  
  - Revisión de código con Serverspec y pruebas de integración continua  
- Revisión de configuraciones contrastando parámetros críticos con estándares de endurecimiento (CIS Benchmarks, OWASP DevSecOps).  
- Uso de listas de verificación para mapear controles y evaluar el nivel de madurez y cumplimiento conforme a ISO 27001, COBIT 2019 y NIST SP 800-53.

---
# 7. HALLAZGOS Y OBSERVACIONES

## 7.1 Seguridad de la Información

**1. Exposición de credenciales sensibles**  
- **Descripción**: Se identificaron variables como `DB_PASSWORD` y `WP_ADMIN_PASS` guardadas en texto plano dentro de data bags de Chef y en el archivo `.env`.  
- **Evidencia**: Captura de pantalla del archivo `data_bag_item/mysql/root.json` y referencia al commit `#3c1f2a7` en el repositorio.  
- **Criticidad**: Alto (25)  
- **Criterio vulnerado**: ISO/IEC 27001:2022 (Control 8.12), NIST SP 800-53 (AC-6), Política de Seguridad de la Información de la organización.  
- **Causa**: Falta de un mecanismo de cifrado de secretos (por ejemplo, Chef Vault o HashiCorp Vault).  
- **Efecto**: Riesgo elevado de accesos no autorizados a la base de datos y al panel de administración de WordPress.

---

**2. Puertos abiertos sin restricciones y falta de firewall**  
- **Descripción**: Las máquinas virtuales se desplegaron en modo `host-only` sin aplicar reglas de firewall como iptables o UFW.  
- **Evidencia**: Resultado de un escaneo `nmap` sobre `192.168.56.0/24` que muestra puertos 22, 80, 443 y 3306 abiertos para cualquier host.  
- **Criticidad**: Alto (20)  
- **Criterio vulnerado**: ISO/IEC 27002:2022 (Control 8.20), CIS Benchmark Ubuntu 22.04, Sección 3.5.  
- **Causa**: Configuración predeterminada de Vagrant sin endurecimiento.  
- **Efecto**: Superficie de ataque incrementada, facilitando movimientos laterales o explotación remota.

---

**3. Falta de registros de auditoría persistentes**  
- **Descripción**: No se habilitaron mecanismos de registro persistente como `rsyslog`, ni se redirigieron logs de `chef-client` a almacenamiento duradero.  
- **Evidencia**: Revisión del `recipe[chef_client]` sin directivas de `log_location`.  
- **Criticidad**: Alto (16)  
- **Criterio vulnerado**: ISO 22301:2019 (Cláusula 8.4), NIST SP 800-53 (AU-6).  
- **Causa**: Se priorizó la agilidad operativa por encima de la trazabilidad.  
- **Efecto**: Dificultad para reconstruir incidentes de seguridad o errores de servicio.

---

**4. Uso de versiones de software obsoletas**  
- **Descripción**: Se detectaron versiones antiguas de Apache 2.4.54, MySQL 5.7 y Ruby 2.6 sin parches recientes.  
- **Evidencia**: Salida de `apachectl -v`, `mysql --version` y referencias a CVE pendientes de mitigación.  
- **Criticidad**: Alto (20)  
- **Criterio vulnerado**: OWASP Top 10 (A06:2021 — Componentes vulnerables), Política de Gestión de Parches TI.  
- **Causa**: No contar con un ciclo de actualización automatizado en los cookbooks.  
- **Efecto**: Mayor exposición ante vulnerabilidades conocidas.

---

## 7.2 Gestión de Cambios y Configuración

**1. Ambiente único sin segmentación (dev/test/prod)**  
- **Descripción**: Se empleó un solo `Vagrantfile` para desarrollo, pruebas y staging, sin distinción de etiquetas o perfiles específicos.  
- **Evidencia**: Revisión de la rama `main` del repositorio sin variables de entorno diferenciadas (`VAGRANT_ENV`).  
- **Criticidad**: Alto (20)  
- **Criterio vulnerado**: COBIT 2019 (BAI03.03), ITIL 4 (Change Enablement).  
- **Causa**: Simplificación del flujo DevOps para ganar velocidad de despliegue.  
- **Efecto**: Riesgo de trasladar configuraciones inseguras o inestables hacia producción.

---

**2. Cobertura limitada de pruebas**  
- **Descripción**: El script `tests.sh` solo valida servicios básicos (HTTP 200, puerto 3306), sin incluir pruebas negativas o de seguridad.  
- **Evidencia**: Resultados de ejecución de `./tests.sh` con 10/10 pruebas exitosas, y análisis Serverspec con 8 de 50 controles recomendados.  
- **Criticidad**: Medio (12)  
- **Criterio vulnerado**: OWASP DevSecOps Maturity Model (Nivel 2), Política de QA DevIA360.  
- **Causa**: Falta de definición de casos de prueba de fallos y ataques.  
- **Efecto**: Posibilidad de que vulnerabilidades lleguen a entornos productivos sin ser detectadas.

---

## 7.3 Continuidad del Negocio

**1. Respaldos manuales y no verificados**  
- **Descripción**: Las copias de seguridad de la base de datos se ejecutan manualmente con `mysqldump`, sin pruebas de restauración documentadas.  
- **Evidencia**: Cron job comentado en `db_backup.sh` y ausencia de registros de restauración en `/var/log/backup`.  
- **Criticidad**: Medio (15)  
- **Criterio vulnerado**: ISO 22301:2019 (Cláusula 8.7), NIST SP 800-53 (CP-9).  
- **Causa**: Recursos limitados asignados a los planes de recuperación ante desastres (DRP) y continuidad de negocio (BCP).  
- **Efecto**: Alta probabilidad de pérdida de datos o prolongación de indisponibilidad en caso de fallos graves.

---
# 8. ANÁLISIS DE RIESGOS

## 8.1 Metodología de valoración
Se aplicó la metodología **OWASP Risk Rating** para evaluar los riesgos identificados a partir de cada hallazgo. El nivel de riesgo se calculó considerando la fórmula **Impacto × Probabilidad**, y se clasificó de la siguiente forma:  
- **Alto**: ≥ 20  
- **Medio**: 10 a 19  
- **Bajo**: ≤ 9

---

## 8.2 Resumen de riesgos identificados

| Nº | Riesgo asociado                                                   | Impacto | Probabilidad | Nivel de riesgo |
|----|-------------------------------------------------------------------|---------|--------------|-----------------|
| 1  | Exposición de credenciales sensibles                              | Alto    | Alta         | Alto (25)       |
| 2  | Puertos abiertos sin restricciones y falta de firewall            | Alto    | Media        | Alto (20)       |
| 3  | Falta de registros de auditoría persistentes                      | Medio   | Alta         | Alto (16)       |
| 4  | Uso de versiones de software obsoletas                            | Alto    | Media        | Alto (20)       |
| 5  | Ambiente único sin segmentación (dev/test/prod)                   | Alto    | Media        | Alto (20)       |
| 6  | Cobertura limitada de pruebas                                     | Medio   | Media        | Medio (12)      |
| 7  | Respaldos manuales y no verificados                               | Medio   | Media        | Medio (15)      |

> **Cuadro 2**: Evaluación de impacto y probabilidad de los riesgos identificados.

---

## 8.3 Matriz de Riesgos

| Riesgo                                         | Causa (Vínculo a Anexo)                                                                                                    | Impacto | Probabilidad | Nivel de riesgo |
|-----------------------------------------------|----------------------------------------------------------------------------------------------------------------------------|---------|--------------|-----------------|
| Credenciales sin cifrado                      | [vagrantfile - credenciales expuestas](./evidencias/vagrantfile%20-%20credenciales%20expuestas.png)                        | Alto    | 90%          | Crítico         |
| Puertos sin restricciones                      | [Puertos sin restricciones](./evidencias/Puertos-sin-restricciones.png)                                                    | Alto    | 80%          | Alto            |
| Falta de registros de auditoría persistentes   | [vagrant-logs](./evidencias/vagrant-logs.png)                                                                              | Medio   | 85%          | Alto            |
| Ambiente único sin segmentación                | [ambiente-unico-sin-segmentar](./evidencias/ambiente-unico-sin-segmentar.png)                                              | Alto    | 80%          | Alto            |
| Cobertura limitada de pruebas                  | [ausencia-pruebas-automatizadas](./evidencias/ausencia-pruebas-automatizadas.png)                                          | Medio   | 70%          | Medio           |
| Respaldos manuales no verificados              | [Respaldos manuales sin verificar](./evidencias/Respaldos-manuales-sin-verificar.png)                                      | Medio   | 70%          | Medio           |
| Estado de despliegue inicial                   | [Vagrant status](./evidencias/Vagrant%20status.png)                                                                        | Bajo    | 50%          | Bajo            |

> **Cuadro 3**: Matriz de riesgos con vínculo a las evidencias almacenadas en la carpeta `/evidencias/`.

---


---
# 9. RECOMENDACIONES

## 9.1 Vínculo hallazgo–recomendación

A continuación se listan las acciones correctivas propuestas, vinculadas a cada hallazgo descrito en la sección 7 y priorizadas según los niveles de riesgo identificados en la sección 8:

| Nº | Recomendación técnica / organizativa                                                                                                              | Norma de referencia / objetivo de control                                  |
|----|--------------------------------------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------|
| 1  | Implementar cifrado de secretos con Chef Vault o HashiCorp Vault; eliminar credenciales en texto plano del repositorio.                         | ISO/IEC 27002 8.12; NIST SP 800-53 SC-28                                     |
| 2  | Configurar iptables/UFW en cada VM y restringir el acceso a puertos 22, 80, 443 y 3306 solo desde rangos autorizados.                            | CIS Benchmark Ubuntu 22.04 3.5; ISO 27002 8.20                               |
| 3  | Habilitar `rsyslog`, establecer rotación de logs y reenviar registros a un SIEM con retención de al menos 90 días.                               | NIST SP 800-53 AU-6; ISO 27002 8.15                                           |
| 4  | Automatizar el ciclo de parches (Chef Infra Client + repositorios apt) y suscribirse a alertas CVE.                                              | OWASP A06; COBIT 2019 BAI04                                                   |
| 5  | Definir perfiles independientes `{dev, test, prod}` en el `Vagrantfile` con variables de entorno y ramas específicas en Git.                      | ITIL 4 Change Enablement; COBIT 2019 BAI03                                    |
| 6  | Ampliar `tests.sh` y Serverspec para incluir pruebas negativas, de inyección y análisis de calidad de infraestructura (linting); integrar SAST/DAST. | OWASP DevSecOps Maturity Model Nivel 3; ISO 27002 8.28                       |
| 7  | Automatizar respaldos diarios con `mysqldump` cifrado y planificar restauraciones de validación trimestral con monitoreo de éxito.               | ISO 22301 8.7; NIST SP 800-53 CP-10                                           |

> **Cuadro 4**: Acciones de mejora propuestas asociadas a cada hallazgo.

---

## 9.2 Prioridad de implementación

- **Inmediata (0–30 días)**: Recomendaciones R1, R2 y R3 (por riesgos críticos).  
- **Corto plazo (1–3 meses)**: Recomendaciones R4 y R5.  
- **Mediano plazo (3–6 meses)**: Recomendaciones R6 y R7.

---
# 10. CONCLUSIONES

1. El entorno **Chef_Vagrant_Wp** ofrece una base funcional adecuada para despliegues rápidos, pero presenta deficiencias relevantes en la gestión de secretos, el endurecimiento de la red y la trazabilidad de la información.  
2. Los controles implementados son parciales: garantizan la disponibilidad del servicio, pero no aseguran de forma suficiente la confidencialidad ni la integridad según los requisitos de ISO/IEC 27001 y la Ley N.º 29733.  
3. Cinco de los siete riesgos identificados alcanzan un nivel Alto, lo que incrementa la probabilidad de incidentes críticos si no se corrigen a la brevedad.  
4. La organización dispone de un equipo DevOps calificado y una cultura orientada al software libre, factores que facilitan la aplicación de las recomendaciones sin incurrir en costos significativos de licenciamiento.  
5. Una vez aplicadas las mejoras priorizadas, se prevé una disminución del riesgo global superior al 80%, con alineación a las buenas prácticas de gobierno y gestión de TI (COBIT 2019) y seguridad de la información (ISO 27001:2022).

---

# 11. PLAN DE ACCIÓN Y SEGUIMIENTO

Se propone el siguiente plan de acción acordado con la organización para mitigar los riesgos detectados. El **Comité de Seguridad de DevIA360** supervisará mensualmente el avance hasta que se cierre cada uno de los puntos:

| Nº | Recomendación vinculada                                                                               | Responsable                     | Fecha comprometida |
|----|------------------------------------------------------------------------------------------------------|---------------------------------|--------------------|
| 1  | Implementar cifrado de secretos (Chef Vault / HashiCorp Vault); eliminar credenciales en texto plano | Equipo DevOps & Seguridad       | 31/07/2025         |
| 2  | Configurar iptables/UFW y restringir puertos expuestos                                               | Equipo DevOps                   | 31/07/2025         |
| 3  | Habilitar rsyslog y centralizar registros en un SIEM con retención mínima de 90 días                 | Seguridad de la Información     | 31/07/2025         |
| 4  | Automatizar el ciclo de parches y suscribirse a alertas CVE                                          | Equipo DevOps                   | 30/09/2025         |
| 5  | Segmentar entornos (dev/test/prod) en Vagrantfile y Git                                              | Equipo DevOps                   | 30/09/2025         |
| 6  | Ampliar `tests.sh` y Serverspec con pruebas negativas y de seguridad; integrar SAST/DAST             | QA & DevOps                     | 31/12/2025         |
| 7  | Automatizar respaldos cifrados y planificar restauraciones trimestrales                              | Infraestructura & DevOps        | 30/09/2025         |

> **Cuadro 6**: Plan de acción definido para la mitigación de los hallazgos.

---
# 📎 ANEXOS Y EVIDENCIAS

A continuación se listan las evidencias correspondientes a los hallazgos documentados en el informe de auditoría, con enlaces directos a la carpeta `/evidencias/` del repositorio:

- **Anexo A**: [Vagrant status](./evidencias/Vagrant%20status.png)  
  *Captura del comando `vagrant status` tras el despliegue.*

- **Anexo B**: [vagrantfile - credenciales expuestas](./evidencias/vagrantfile%20-%20credenciales%20expuestas.png)  
  *Evidencia de credenciales sensibles sin cifrar en el Vagrantfile.*

- **Anexo C**: [Puertos sin restricciones](./evidencias/Puertos-sin-restricciones.png)  
  *Visualización de la configuración de puertos abiertos sin restricciones.*

- **Anexo D**: [vagrant-logs](./evidencias/vagrant-logs.png)  
  *Evidencia de la ausencia de logs persistentes durante la provisión.*

- **Anexo E**: [ambiente-unico-sin-segmentar](./evidencias/ambiente-unico-sin-segmentar.png)  
  *Uso de un solo entorno sin segmentación (dev/test/prod).*

- **Anexo F**: [ausencia-pruebas-automatizadas](./evidencias/ausencia-pruebas-automatizadas.png)  
  *Cobertura limitada de pruebas automáticas en el script de validación.*

- **Anexo G**: [Respaldos manuales sin verificar](./evidencias/Respaldos-manuales-sin-verificar.png)  
  *Copia de seguridad manual sin validación ni restauración comprobada.*

---
