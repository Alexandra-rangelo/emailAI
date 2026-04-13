# emltriage

**Plataforma web de análisis forense de correo electrónico para equipos DFIR/SOC**

emltriage es una herramienta de análisis de correo electrónico orientada a Digital Forensics and Incident Response (DFIR). Combina una interfaz web interactiva con un backend FastAPI para extraer artefactos, enriquecer indicadores de compromiso (IOCs) con inteligencia de amenazas, consultar IA y generar reportes forenses editables y descargables en múltiples formatos.

---

## Características principales

- **Análisis determinístico** de archivos `.eml` y `.msg`: cabeceras, cuerpo, adjuntos, URLs, IOCs, autenticación SPF/DKIM/DMARC y trayectoria de enrutamiento.
- **Enriquecimiento CTI**: DNS/WHOIS automático, VirusTotal (con API key) y URLhaus (sin API key).
- **Análisis IA**: soporte para Ollama (local), OpenAI, Anthropic, DeepSeek, Groq y cualquier endpoint compatible con OpenAI.
- **Generador de Reportes IQSEC**: previsualización editable en pantalla con exportación a HTML, Word (.doc) y PDF.
- **Interfaz web completa** sin necesidad de usar la línea de comandos para el flujo principal.

---

## Requisitos

- Python 3.11 o superior
- `pip` y `venv`
- `dig` (para resolución DNS en CTI) — incluido en macOS/Linux por defecto

### Opcionales (según funcionalidades deseadas)

| Funcionalidad | Requisito |
|---|---|
| IA local | [Ollama](https://ollama.com) instalado y corriendo |
| IA en la nube | API key de OpenAI, Anthropic, DeepSeek o Groq |
| Reputación avanzada | API key de VirusTotal |

---

## Instalación

### 1. Clonar el repositorio

```bash
git clone https://github.com/Player0xA/AIEML.git
cd AIEML/emltriage
```

### 2. Crear entorno virtual e instalar dependencias

```bash
# Crear y activar entorno virtual
python3 -m venv venv
source venv/bin/activate        # macOS / Linux
# venv\Scripts\activate         # Windows

# Instalar dependencias base
pip install -r requirements.txt
pip install -e .

# Instalar dependencias del servidor web
pip install fastapi uvicorn[standard]
```

### 3. (Opcional) Instalar soporte de IA en la nube

```bash
pip install -r requirements-ai.txt
```

### Alternativa con Makefile

```bash
make install        # Instalación base
make install-ai     # Con soporte de IA (OpenAI, Anthropic, Ollama)
make install-dev    # Con dependencias de desarrollo y pruebas
```

---

## Levantar el servidor

```bash
# Con el entorno virtual activado:
source venv/bin/activate

# Opción A — directamente con uvicorn
uvicorn web.server:app --host 0.0.0.0 --port 8080 --reload

# Opción B — ejecutando el módulo directamente
python web/server.py
```

Luego abre en el navegador: **http://localhost:8080**

> El flag `--reload` recarga automáticamente el servidor al guardar cambios en el código (útil durante desarrollo).

---

## Uso de la interfaz web

### Panel: Análisis de Correo

1. Arrastra o selecciona un archivo `.eml` o `.msg`.
2. (Opcional) ingresa una API key de VirusTotal para enriquecimiento avanzado.
3. Haz clic en **Analizar** — el backend procesa el correo y devuelve los artefactos.

### Panel: Inteligencia de Amenazas (CTI)

- Se ejecuta automáticamente tras el análisis.
- Realiza consultas DNS y WHOIS en paralelo para todos los dominios encontrados.
- Si se proporcionó una API key de VirusTotal, consulta reputación de dominios, IPs y URLs.
- Consulta URLhaus sin necesidad de API key.

### Panel: Resumen IA

- Configura el endpoint de tu LLM preferido (Ollama local, OpenAI, DeepSeek, Groq, etc.).
- Selecciona el modelo y la longitud del análisis (rápido, medio, completo).
- Genera un análisis narrativo del correo basado en los artefactos extraídos.

### Panel: Generador de Reportes (Report Builder)

1. **Generar Reporte**: genera una vista previa editable con los datos del análisis en formato de reporte forense IQSEC.
2. **Edición en pantalla**: todos los campos de texto son editables directamente. Se pueden:
   - Editar texto de cualquier campo libre.
   - Eliminar secciones completas (ícono `×` al hacer hover sobre el título `h2`).
   - Eliminar subsecciones completas (ícono `×` al hacer hover sobre el subtítulo `h3`).
   - Eliminar filas de cualquier tabla (ícono `×` al final de cada fila al hacer hover).
   - Eliminar columnas de cualquier tabla (ícono `×` en la esquina del encabezado al hacer hover).
   - Pegar capturas de pantalla con `Ctrl+V` / `Cmd+V` en la sección de evidencias.
3. **Exportar**:
   - **Descargar HTML**: descarga el reporte como archivo HTML autocontenido.
   - **Descargar Word**: descarga en formato `.doc` compatible con Microsoft Word.
   - **Descargar PDF**: abre el diálogo de impresión del navegador para guardar como PDF.

---

## Configuración de API keys

Copia el archivo de ejemplo y edítalo:

```bash
cp .env.example .env
```

Contenido del `.env`:

```env
# IA local (Ollama — sin API key)
OLLAMA_BASE_URL=http://localhost:11434

# IA en la nube (opcional)
OPENAI_API_KEY=sk-tu-clave-aqui
ANTHROPIC_API_KEY=sk-ant-tu-clave-aqui

# Inteligencia de amenazas (opcional)
VIRUSTOTAL_API_KEY=tu-clave-vt-aqui
ABUSEIPDB_API_KEY=tu-clave-abuseipdb-aqui

# Proveedor IA por defecto
EMLTRIAGE_AI_PROVIDER=ollama
```

> La herramienta funciona **100% offline** sin ninguna API key. Las claves solo se necesitan para funcionalidades en la nube.

---

## Configurar Ollama (IA local)

```bash
# Instalar Ollama (macOS/Linux)
curl -fsSL https://ollama.com/install.sh | sh

# Descargar un modelo (ej. llama3.1)
ollama pull llama3.1

# Iniciar el servidor Ollama
ollama serve
```

En la interfaz web, configura el endpoint como `http://localhost:11434` y el modelo como `llama3.1`.

---

## Uso por línea de comandos (CLI)

El CLI está disponible para flujos automatizados o integración con otras herramientas:

```bash
# Análisis básico
emltriage analyze email.eml -o ./output

# Análisis profundo (detección de macros)
emltriage analyze suspicious.eml -o ./output --mode deep

# Procesamiento en lote
emltriage batch ./emails/ -o ./results

# Enriquecimiento CTI
emltriage cti ./output/iocs.json -o ./output --watchlist ./watchlists/ --online

# Análisis IA (requiere Ollama corriendo)
emltriage ai ./output/artifacts.json -o ./output --cti ./output/cti.json
```

### Estructura de salida del CLI

```
output/
├── artifacts.json          # Artefactos completos extraídos
├── iocs.json               # IOCs normalizados y filtrados
├── auth_results.json       # Resultados SPF/DKIM/DMARC
├── report.md               # Reporte determinístico en Markdown
├── manifest.json           # Hashes SHA256 de archivos
├── cti.json                # Enriquecimiento CTI (fase 2)
├── ai_report.json          # Análisis IA estructurado (fase 3)
├── ai_report.md            # Narrativa IA en Markdown
├── attachments/            # Adjuntos extraídos
│   ├── <id>_documento.pdf
│   └── <id>_imagen.png
├── body_1.txt              # Cuerpo en texto plano
└── body_2.html             # Cuerpo HTML decodificado
```

---

## Arquitectura del proyecto

```
emltriage/
├── web/
│   ├── server.py           # Backend FastAPI (API REST + archivos estáticos)
│   ├── index.html          # Interfaz web principal
│   ├── app.js              # Lógica frontend completa
│   ├── styles.css          # Estilos generales de la UI
│   └── report-preview.css  # Estilos del generador de reportes
├── emltriage/
│   ├── cli.py              # CLI con Typer (5 comandos)
│   ├── core/               # Fase 1: Extracción determinística
│   │   ├── parser.py       # Orquestador principal
│   │   ├── models.py       # Esquemas Pydantic (30+)
│   │   ├── ioc_filter.py   # Filtrado de ruido de infraestructura
│   │   └── extract/        # Módulos de extracción
│   ├── cti/                # Fase 2: Enriquecimiento CTI
│   │   ├── engine.py       # Orquestador CTI
│   │   ├── cache.py        # Caché SQLite con TTL
│   │   └── providers/      # VirusTotal, AbuseIPDB, URLhaus, watchlists
│   ├── ai/                 # Fase 3: Narrativa IA
│   │   ├── engine.py       # Orquestador IA
│   │   ├── validators.py   # Disciplina de evidencia / anti-alucinación
│   │   └── providers/      # Ollama, OpenAI, Anthropic
│   ├── reporting/          # Generación de reportes
│   │   ├── docx/           # Renderer Word
│   │   └── json_generator.py
│   └── infra/
│       └── robust_whois.py # WHOIS con fallback a RDAP
├── requirements.txt        # Dependencias base
├── requirements-ai.txt     # Dependencias IA en la nube
├── pyproject.toml          # Configuración del paquete
├── Makefile                # Comandos de instalación y desarrollo
└── .env.example            # Plantilla de variables de entorno
```

---

## API REST del backend

El servidor expone los siguientes endpoints:

| Método | Endpoint | Descripción |
|---|---|---|
| `POST` | `/api/analyze` | Analiza un archivo `.eml` o `.msg` |
| `POST` | `/api/cti/fast` | DNS + WHOIS en paralelo para dominios |
| `POST` | `/api/cti/vt` | Reputación VirusTotal |
| `POST` | `/api/cti/urlhaus` | Reputación URLhaus (sin API key) |
| `POST` | `/api/ai/summarize` | Genera resumen IA vía endpoint configurable |
| `GET`  | `/api/ai/status` | Estado del proveedor IA (Ollama) |
| `POST` | `/api/export/docx` | Exporta reporte como `.docx` |
| `POST` | `/api/report/generate` | Genera reporte JSON con narrativa IA |

---

## Desarrollo

```bash
# Instalar dependencias de desarrollo
make install-dev

# Ejecutar pruebas
make test

# Linter
make lint

# Formatear código
make format

# Verificación de tipos
make typecheck

# Limpiar artefactos de build
make clean
```

---

## Solución de problemas

### El servidor no inicia

```bash
# Verificar que el entorno virtual esté activado
source venv/bin/activate

# Verificar que fastapi y uvicorn estén instalados
pip install fastapi uvicorn[standard]

# Verificar que el puerto 8080 esté libre
lsof -i :8080
```

### Error de versión de Python

```bash
python3 --version   # Debe ser 3.11 o superior

# Si es más antigua, usar pyenv:
pyenv install 3.11
pyenv local 3.11
```

### Ollama no disponible

```bash
# Verificar que Ollama esté corriendo
curl http://localhost:11434/api/tags

# Iniciar si no está activo
ollama serve

# Descargar modelo si es necesario
ollama pull llama3.1
```

### Error al analizar archivos .msg (Outlook)

```bash
# Asegurar que extract-msg esté instalado
pip install extract-msg>=0.55.0
```

---

## Licencia

MIT License — ver archivo [LICENSE](./LICENSE)

---

**¿Listo para analizar correos sospechosos?**
Instala, levanta el servidor con `uvicorn web.server:app --port 8080` y abre `http://localhost:8080` en tu navegador.
