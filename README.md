<!--
  README.md — DRBOT · El Asistente Médico
  Grupo de DRBOT · Proyecto de Informática II
  Última actualización: Junio 2026
-->

<div align="center">

# 🩺 DRBOT · El Asistente Médico

### Sistema de asistencia médica virtual basado en Inteligencia Artificial

**Optimiza la fase previa a la consulta médica recopilando, organizando y resumiendo la información clínica del paciente.**

[![Java](https://img.shields.io/badge/Java-8-orange?logo=openjdk&logoColor=white)](https://adoptium.net/temurin/releases/?version=8)
[![Maven](https://img.shields.io/badge/Maven-3.6%2B-C71A36?logo=apache-maven&logoColor=white)](https://maven.apache.org/)
[![MySQL](https://img.shields.io/badge/MySQL-8.0-4479A1?logo=mysql&logoColor=white)](https://dev.mysql.com/downloads/)
[![Groq](https://img.shields.io/badge/Groq-LLaMA%203.3--70B-FF6B35?logo=ai&logoColor=white)](https://console.groq.com/)
[![ESP32](https://img.shields.io/badge/ESP32-Biometr%C3%ADa-000000?logo=espressif&logoColor=white)](https://www.espressif.com/)
[![License](https://img.shields.io/badge/License-Private-red.svg)](#-licencia)
[![Status](https://img.shields.io/badge/Status-Prototipo-yellow.svg)](#-estado-del-proyecto)

[Descripción](#-descripción) ·
[Características](#-características-principales) ·
[Arquitectura](#-arquitectura) ·
[Instalación](#-instalación-rápida) ·
[Uso](#-uso) ·
[Roadmap](#-roadmap) ·
[Equipo](#-equipo)

</div>

---

## 📋 Descripción

**DRBOT** es una aplicación de escritorio desarrollada en **Java + Swing** que actúa como **asistente médico virtual basado en IA**. Su objetivo es resolver uno de los problemas principales de la atención sanitaria: **la saturación de consultas y la lentitud en la recogida de información clínica del paciente**.

El sistema permite al paciente **introducir sus síntomas antes de la cita**, los procesa mediante un modelo de lenguaje (LLaMA 3.3-70B vía Groq API) y **genera un informe clínico estructurado** que el médico recibe antes de iniciar la consulta. De este modo, el profesional llega a la cita con un resumen claro de la situación del paciente, reduciendo el tiempo de anamnesis y mejorando la calidad de la atención.

> ⚠️ **DRBOT no reemplaza al médico.** Es una herramienta de **apoyo a la decisión clínica**. La responsabilidad del diagnóstico recae exclusivamente en el profesional sanitario.

**Entornos objetivo:** clínicas privadas y hospitales públicos, bajo modelo de licencia por contrato adaptado a cada institución.

---

## ✨ Características principales

### 👤 Para el paciente
- 🔐 Registro y autenticación segura con roles diferenciados.
- 📝 **Toma de síntomas guiada** antes de la consulta (texto libre 50–950 caracteres).
- 🧠 **Análisis con IA** que genera un informe clínico estructurado.
- 📅 **Solicitud de citas** con calendario semanal interactivo por especialidad y médico.
- 📜 Consulta del **historial clínico** completo con detalle de cada consulta.
- 🖼️ **Foto de perfil** capturada desde webcam (OpenCV en macOS / Webcam-capture en Windows).
- ⚙️ **Configuración personalizable:** 7 idiomas, modo oscuro/claro, notificaciones.

### 🩺 Para el médico
- 🔒 **Autenticación biométrica** con huella dactilar (ESP32 + sensor AS608/R307) además de contraseña.
- 📊 **Panel de gestión centralizado:** pacientes, calendario, historial, diagnósticos.
- 🤖 Revisión del **informe preliminar generado por IA** antes de la consulta.
- ✍️ Registro de **diagnósticos y comentarios clínicos** con área de texto enriquecida (undo/redo, atajos de teclado).
- 📆 Vista semanal de citas con franjas horarias codificadas por colores.

### 🛡️ Seguridad
- 🔑 Contraseñas con **hash SHA-256**.
- 🖐️ Huellas dactilares almacenadas como **hash SHA-256 con sal** (nunca en texto plano).
- 🧱 **PreparedStatement** en el 100% de las consultas SQL (prevención de inyección).
- 🎭 **Control de acceso basado en roles** (RBAC).

### 🌐 Internacionalización
- 7 idiomas soportados: 🇪🇸 español, 🇬🇧 inglés, 🇫🇷 francés, 🇩🇪 alemán, 🇮🇹 italiano, 🇵🇹 portugués, 🇮🇳 hindi.
- Cambio de idioma **dinámico** sin reiniciar la aplicación.

---

## 🏗️ Arquitectura

DRBOT sigue una **arquitectura de cliente pesado en 4 capas**:
┌─────────────────────────────────────────────────────────┐
│ CAPA DE PRESENTACIÓN (Swing + Nimbus LookAndFeel) │
│ FrameInicioSesion · FramePaciente · FrameMedico │
│ Paneles · Diálogos modales · Componentes reutilizables │
├─────────────────────────────────────────────────────────┤
│ CAPA DE LÓGICA DE NEGOCIO │
│ Usuario · Medic · DatosUsuario · AjustesUsuario │
│ ConsultaMedicaGroq · TextoManager (i18n) │
├─────────────────────────────────────────────────────────┤
│ CAPA DE PERSISTENCIA (DAO + JDBC) │
│ UsuarioDAO · ConsultaDAO · ConexionBBDD (singleton) │
├─────────────────────────────────────────────────────────┤
│ CAPA DE INFRAESTRUCTURA │
│ ChatIAGroq (HTTP → Groq) · ComunicacionESP32 (serie) │
│ DialogoCapturaWebcam (OpenCV) · HashUtil · TratoImagen │
└─────────────────────────────────────────────────────────┘

text

### 🧠 Módulo de IA clínica

El módulo `ConsultaMedicaGroq` construye un **prompt médico estructurado** que instruye al modelo LLaMA 3.3-70B para generar un informe con:

1. **Síntomas clave** extraídos del texto libre.
2. **Cronología** de la evolución.
3. **Factores agravantes y atenuantes.**
4. **Preguntas clave** que el médico debería formular al paciente.
5. **Señales de alerta** a monitorizar.

> El resultado se almacena en `consultations.patient_symptoms_ai` y es visible para el médico en su panel.

### 🗄️ Base de datos

10 tablas relacionales en MySQL 8 (`drbot_mysql`) con claves foráneas en `CASCADE`:

| Tabla | Descripción |
|---|---|
| `users` | Credenciales y rol (paciente, medico, admin, recepcion) |
| `users_data` | Datos personales y clínicos (1:1) |
| `users_config` | Preferencias visuales y de idioma (1:1) |
| `medics_data` | Especialidad, nº colegiación, huella hasheada (1:1) |
| `medic_schedules` | Horario semanal por médico (N:1) |
| `consultations` | Citas y consultas (síntomas raw + IA + diagnóstico) |
| `users_diagnostics` | Diagnósticos emitidos por médicos |
| `users_alergies` | Alergias normalizadas (N:1) |
| `users_meds` | Medicamentos normalizados (N:1) |
| `medic_fingerprints` | Historial de huellas biométricas |

---

## 🛠️ Stack tecnológico

| Capa | Tecnología |
|---|---|
| **Lenguaje** | Java 8 (JDK 1.8) |
| **Build** | Apache Maven 3.6+ |
| **UI** | Java Swing + Nimbus LookAndFeel personalizado |
| **BBDD** | MySQL 8.0 + MySQL Connector/J 8.4.0 |
| **IA** | Groq API · LLaMA 3.3-70B (temp. 0.6, máx. 800 tokens) |
| **Cliente HTTP** | OkHttp 4.11.0 + GSON |
| **Hardware** | ESP32 + sensor de huella AS608/R307 |
| **Serial** | jSerialComm + jSSC |
| **Visión** | OpenCV/JavaCV (macOS/Linux) · Webcam-capture (Windows) |
| **Calidad** | Checkstyle (Google Java Style) · Prettier · EditorConfig · SonarQube |
| **Gestión** | Git + GitHub · Trello · Actas en Drive |

---

## 🚀 Instalación rápida

### Requisitos previos

| Componente | Versión mínima | Notas |
|---|---|---|
| **JDK** | 8 (1.8) | Compatible hasta Java 11. No usar Java 21+ sin verificar JavaCV |
| **Maven** | 3.6+ | Gestor de dependencias y build |
| **MySQL** | 8.0+ | Base de datos principal |
| **Arduino IDE** | 2.0+ | Solo si vas a flashear el ESP32 (opcional) |

**Hardware opcional:** ESP32 con sensor de huellas AS608 · Webcam USB.

### 1️⃣ Clonar el repositorio

```bash
git clone https://github.com/xDit0x/JAVADrBotMAVEN.git drbotprogram
cd drbotprogram
🔒 El repositorio es privado. Solicita acceso contactando con los desarrolladores.

2️⃣ Configurar la base de datos
sql
CREATE DATABASE IF NOT EXISTS drbot_mysql
  CHARACTER SET utf8mb4
  COLLATE utf8mb4_0900_ai_ci;
bash
mysql -u root -p drbot_mysql < database/tablesDrBOT.sql
3️⃣ Configurar credenciales
bash
cp database.env.ejemplo database.env
cp apiKey.env.ejemplo apiKey.env
Edita database.env:

env
db.url=jdbc:mysql://localhost:3306/drbot_mysql?useSSL=false&serverTimezone=UTC
db.user=root
db.password=tu_contraseña
db.driver=com.mysql.cj.jdbc.Driver
Edita apiKey.env (obtén tu clave en console.groq.com):

env
GROQ_API_KEY=gsk_tu_api_key_real_aqui
GROQ_MODEL=llama-3.3-70b-versatile
DEFAULT_TEMPERATURE=0.6
MAX_TOKENS=800
⚠️ Añade database.env y apiKey.env a tu .gitignore. Nunca los subas al repositorio.

4️⃣ Compilar y ejecutar
bash
# Compilar
mvn clean compile

# Ejecutar en desarrollo
mvn compile exec:java -Dexec.mainClass="com.drbotprogram.App"

# Generar JAR ejecutable
mvn clean package
java -jar target/drbotprogram-1.0-SNAPSHOT.jar
5️⃣ Usuarios por defecto
Usuario	Contraseña	Rol
testPatient	test	Paciente
testMedic	test	Médico
admin	admin	Administrador
dr_smith	medico123	Médico
6️⃣ Configuración del ESP32 (opcional, solo médicos)
Pin AS608	Pin ESP32
TX	GPIO27
RX	GPIO26
VCC	3.3V
GND	GND
Abre Sistema_biometrico_DRBOT/Sistema_biometrico_DRBOT.ino en Arduino IDE.

Instala la librería Adafruit Fingerprint Sensor Library.

Selecciona placa ESP32 Dev Module y puerto correcto.

Sube el código (USB 115200 baud, UART sensor 57600 baud).

🎬 Uso
Flujo del paciente
Registrarse o iniciar sesión como paciente.

Solicitar cita → seleccionar especialidad → médico → franja horaria.

Antes de la cita: acceder a Previo Consulta → describir síntomas (50–950 caracteres) → Enviar.

La IA genera un informe estructurado y lo almacena.

Consultar historial de consultas anteriores con detalle completo.

Flujo del médico
Iniciar sesión con alias + contraseña → verificación biométrica con huella.

Ver panel de pacientes con búsqueda por nombre/alias.

Acceder a consulta actual: revisar síntomas raw + análisis IA → escribir diagnóstico → Guardar y siguiente.

Consultar calendario semanal y detalles de pacientes (4 pestañas: datos, historial, contacto, consultas).

📁 Estructura del proyecto
text
drbotprogram/
├── pom.xml                          # Configuración Maven y dependencias
├── database.env                     # Credenciales MySQL (NO SUBIR A GIT)
├── apiKey.env                       # API Keys (NO SUBIR A GIT)
├── database/
│   └── tablesDrBOT.sql              # DDL + datos de ejemplo
├── Sistema_biometrico_DRBOT/
│   └── Sistema_biometrico_DRBOT.ino # Firmware Arduino ESP32
├── src/main/java/com/drbotprogram/
│   ├── App.java                     # Punto de entrada (singleton)
│   ├── Components/                  # Capa de lógica de negocio
│   │   ├── ChatIA/                  # ChatIAGroq · ConsultaMedicaGroq
│   │   ├── Consulta/                # Consulta · ConsultaDAO · Especialidad
│   │   ├── ESP32Coms/               # ComunicacionESP32 · HashUtil
│   │   ├── Util/                    # TratoImagen · IdiomaUtil · Borders · Labels
│   │   ├── ConexionBBDD.java
│   │   ├── ConstantesDrBot.java
│   │   └── UIManagerConfig.java
│   ├── Frames/
│   │   ├── Dialogs/                 # FrameConsulta · DialogHuella · ...
│   │   └── MainFrames/              # FrameInicioSesion · FramePaciente · FrameMedico
│   ├── Languages/                   # mensajes_{es,en,fr,de,it,pt,hi}.properties
│   └── Usuario/                     # Usuario · Medic · DatosUsuario · ...
└── .vscode/
    ├── launch.json
    └── settings.json
🧪 Testing y calidad
Herramientas configuradas
Checkstyle (Google Java Style) en fase validate de Maven.

EditorConfig (.editorconfig) para indentación consistente (2 espacios).

Prettier (.prettierrc) — tabWidth: 2, useTabs: false.

SonarQube (sonar-maven-plugin 3.11.0) para análisis continuo.

Estado de la suite de tests
⚠️ Actualmente no existe suite automatizada de tests. El plan de pruebas definido en la memoria cubre:

Caja negra: validación funcional (login, registro, citas, IA, consulta, webcam, ESP32).

Caja blanca: cobertura de caminos (i18n, DAO, TratoImagen, LabelAnimado).

Validación: cumplimiento de requisitos funcionales (FRQ-01 a FRQ-15).

Estrés/rendimiento: 50 logins concurrentes, 10 peticiones Groq, 100 capturas webcam, 100 ciclos ESP32.

Seguridad: inyección SQL, acceso sin autenticación, protección de API key, hash de huellas.

La implementación de la suite de tests queda como trabajo futuro (ver Roadmap).

🗺️ Roadmap
✅ Completado
☑ Autenticación con roles (paciente, médico, admin, recepción).
☑ Autenticación biométrica con ESP32 + huella dactilar.
☑ Toma de síntomas + análisis con IA (Groq/LLaMA 3.3-70B).
☑ Panel de gestión médica (pacientes, calendario, historial, diagnósticos).
☑ Gestión de citas con calendario semanal interactivo.
☑ Internacionalización (7 idiomas) con cambio dinámico.
☑ Modo oscuro/claro y configuración de notificaciones.
☑ Captura de foto de perfil con webcam.
☑ Hash SHA-256 con sal para contraseñas y huellas.
🚧 En desarrollo / Corto plazo
□ Asistente conversacional con IA propia (preguntas adaptativas).
□ Modelo 3D del cuerpo humano para localizar el dolor.
□ Hacer funcional el cambio de tamaño de letra.
□ Completar las 296 claves de traducción en los 7 idiomas.
🔮 Medio plazo
□ IA propia entrenada con datos clínicos certificados (o migración a GPT-4.1 / Azure OpenAI con cumplimiento HIPAA).
□ Cumplimiento RGPD + HIPAA (cifrado en reposo y tránsito, logs de auditoría inalterables, BAA con proveedores).
□ Gestor de secretos en lugar de archivos .env.
□ Suite de tests automatizada (JUnit + Surefire).
□ Refactorización arquitectónica: capa MVC/Servicio, principios SOLID, HikariCP para pool de conexiones, SLF4J/Logback.
□ División de UsuarioDAO (862 líneas) en DAOs especializados.
🌐 Largo plazo
□ Arquitectura web (Spring Boot + Angular/React) o híbrida Swing + web.
□ Integración con HCE hospitalarias mediante HL7 FHIR.
□ Cifrado TDE en MySQL + backups automatizados.
□ Docker + Kubernetes para escalabilidad horizontal.
□ Certificación ISO 13485 (productos sanitarios).
🤝 Contribuir
Este es un repositorio privado de un proyecto académico (Proyecto de Informática II). Si formas parte del equipo:

Crea una rama: git checkout -b feature/nueva-funcionalidad

Sigue el estilo de código (Google Java Style vía Checkstyle).

Ejecuta mvn clean compile antes de hacer commit.

Abre un Pull Request describiendo los cambios.

Solicita revisión a otro miembro del equipo.

Convención de commits: tipo(alcance): descripción — ej. feat(auth): añadir verificación biométrica, fix(webcam): resolver SIGSEGV en macOS.

⚠️ Limitaciones conocidas
Área	Estado
Cumplimiento RGPD/HIPAA	❌ No implementado (prototipo no apto para entorno clínico real)
Chatbot conversacional	❌ No implementado (solo texto libre)
Modelo 3D del cuerpo	❌ No implementado
Cambio de tamaño de letra	❌ Interfaz lista, sin efecto real
Traducciones i18n	⚠️ 7 idiomas definidos, no todos completos (296 claves)
Arquitectura SOLID/SRP	⚠️ Sin capa MVC/Servicio, UI acoplada a BD
Suite de tests	❌ No implementada
Pool de conexiones	❌ Sin HikariCP
Logging	⚠️ Sin SLF4J, 48 catch (Exception e) genéricos
👥 Equipo
<div align="center">
👨‍💻 Desarrollador	Rol
Diego Andrés Domínguez	Desarrollo full stack · IA · Integración
Pedro Martínez	Desarrollo full stack · BBDD · UI
Sebastián Escobar	Desarrollo full stack · Hardware · QA
</div>
Institución: Proyecto de Informática II · Curso 2025–2026
Fecha de entrega: 5 de junio de 2026

📄 Licencia
Este proyecto es privado y de uso académico. Todos los derechos reservados © 2026 · Grupo DRBOT.

El código no puede ser copiado, distribuido ni modificado sin autorización expresa de los autores. Para consultas sobre licencia, contacta con el equipo.

🙏 Agradecimientos
Grupo APALABRAZOS por la auditoría externa de calidad de código.

Docentes de Proyecto de Informática II por el seguimiento y las directrices.

Comunidad open source: OpenCV, JavaCV, OkHttp, GSON, jSerialComm, Adafruit Fingerprint Library.

Groq por el acceso a la API de LLaMA 3.3-70B.

<div align="center">
⬆ Volver arriba

Hecho con ❤️ y mucho ☕ por el equipo de DRBOT

</div> ```
