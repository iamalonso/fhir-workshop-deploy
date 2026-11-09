# fhir-workshop-deploy

Entorno de trabajo del curso **Implementación de Ecosistemas Digitales Interoperables en Salud** · Unidad 3: Implementación de servicios interoperables con HL7 FHIR.

Este repositorio levanta un servidor **HAPI FHIR R4** en tu computador con Docker, y trae una colección de Postman para consumirlo. Lo usaremos desde la Sesión 7 hasta el cierre de la unidad.

```
Postman ──HTTP──▶ HAPI FHIR R4 (contenedor Docker) ──▶ Base de datos H2 (volumen)
   │
   └── colección y entorno versionados en GitHub
```

---

## Laboratorio: Sesión 7

**Objetivo:** terminar con un servidor FHIR R4 funcionando, verificado y versionado, listo para crear recursos en la Sesión 8.

**Entregable:** link a tu repo con
1. la captura del CapabilityStatement de tu servidor local, y
2. las 2 diferencias encontradas en el paso F,

ambos dentro de la carpeta `entregables/`.

### Prerrequisitos

| Herramienta | Para qué | Verificación |
|---|---|---|
| [Docker Desktop](https://www.docker.com/products/docker-desktop/) | Correr el servidor en un contenedor | `docker --version` |
| [Git](https://git-scm.com/downloads) | Clonar y versionar el repo | `git --version` |
| [Postman](https://www.postman.com/downloads/) (app de escritorio) | Enviar requests al servidor | Abre la app |
| Cuenta de [GitHub](https://github.com/) | Hacer fork y entregar | Inicia sesión |

> **Windows:** Docker Desktop necesita **WSL 2**. Si al instalar te lo pide, acepta y reinicia.
>
> **Recursos:** el servidor usa cerca de 2 GB de RAM. Cierra aplicaciones pesadas si tu equipo tiene 8 GB o menos.

---

### A: Instalar

Instala las cuatro herramientas de la tabla y verifica en una terminal:

```bash
docker --version
docker compose version
git --version
```

Abre Docker Desktop y espera a que indique que está corriendo (*Engine running*).

### B: Hacer fork y clonar

1. En GitHub, presiona **Fork** sobre este repositorio para tener tu propia copia.
2. Clónala en tu computador (reemplaza `TU-USUARIO`):

```bash
git clone https://github.com/TU-USUARIO/fhir-workshop-deploy.git
cd fhir-workshop-deploy
```

### C: Levantar el servidor

```bash
docker compose up -d
```

La primera vez descarga la imagen (~400 MB) y el servidor tarda **1 a 3 minutos** en arrancar. Para ver si ya está listo:

```bash
docker compose logs -f fhir
```

Cuando aparezca una línea con `Started Application`, presiona `Ctrl + C` para salir de los logs (el servidor sigue corriendo).

```bash
docker ps        # debe aparecer el contenedor hapi-fhir-r4
```

### D: Verificar el servidor

Abre en el navegador:

```
http://localhost:8080/fhir/metadata
```

Debes ver un **CapabilityStatement**. Busca estos elementos:

| Elemento | Valor esperado | Qué significa |
|---|---|---|
| `resourceType` | `CapabilityStatement` | Es la declaración de capacidades del servidor |
| `fhirVersion` | `4.0.1` | El servidor habla FHIR R4 |
| `software.name` | HAPI FHIR Server | La implementación que estamos usando |
| `rest[0].resource` | Lista de tipos de recurso | Qué recursos acepta y qué interacciones permite en cada uno |

📸 Toma una captura donde se vea `fhirVersion` y guárdala como `entregables/capability-local.png`.

También puedes abrir `http://localhost:8080/` para ver la interfaz web del servidor.

### E: Configurar Postman

1. En Postman: **Import** → selecciona los dos archivos de la carpeta `postman/`.
2. Arriba a la derecha, elige el entorno **FHIR local (HAPI R4)**.
3. Abre la colección **FHIR Workshop / Sesión 7** y ejecuta **01 · CapabilityStatement (local)**.
4. En la pestaña **Test Results** deben pasar los 3 tests.
5. Ejecuta **02** y compara: es la misma información, ahora en XML.

> La variable `{{baseUrl}}` evita escribir la dirección del servidor en cada request. Si mañana cambias de servidor, solo cambias el entorno.

### F: Comparar con un servidor público

Ejecuta **03 · CapabilityStatement (servidor público)**, que apunta a `https://hapi.fhir.org/baseR4`.

Compara ambas respuestas y anota **2 diferencias** en `entregables/diferencias.md` (hay una plantilla en `entregables/README.md`). Algunas pistas de dónde mirar:

- `software.version` y `implementation`
- `rest[0].resource`: ¿cuántos tipos de recurso y qué interacciones?
- `format`: ¿qué formatos acepta cada uno?
- `date`

> El servidor público es compartido y se reinicia periódicamente: **nunca subas datos reales de pacientes**.

### G: Versionar y entregar

```bash
git add entregables/
git commit -m "S7: entorno FHIR R4 verificado"
git push
```

Entrega el link de tu repo en la plataforma del curso.

---

## Comandos útiles

| Acción | Comando |
|---|---|
| Levantar | `docker compose up -d` |
| Ver logs | `docker compose logs -f fhir` |
| Detener (conserva datos) | `docker compose down` |
| Reiniciar | `docker compose restart` |
| Borrar todo, incluidos los datos | `docker compose down -v` ⚠️ |

Los datos viven en el volumen `hapi-data`. Detener o reiniciar el servidor **no** los borra; solo `down -v` lo hace. Esto importa porque en las sesiones 8 a 11 seguirás trabajando sobre este mismo servidor.

## Errores frecuentes

| Síntoma | Causa probable | Solución |
|---|---|---|
| `port is already allocated` o `8080 ... in use` | Otro programa usa el puerto 8080 | En `docker-compose.yml` cambia `"8080:8080"` por `"8081:8080"` y usa `http://localhost:8081/fhir` (actualiza también `baseUrl` en Postman) |
| `Cannot connect to the Docker daemon` | Docker Desktop no está abierto | Abre Docker Desktop y espera a *Engine running* |
| Docker pide WSL 2 (Windows) | WSL no está activado | Abre PowerShell como administrador: `wsl --install` y reinicia |
| El navegador no carga `/metadata` | El servidor aún está arrancando | Espera y revisa `docker compose logs -f fhir` |
| El contenedor se detiene solo | Memoria insuficiente | Docker Desktop → Settings → Resources: asigna al menos 4 GB |
| Postman: `Could not get response` | `baseUrl` mal escrita o entorno no seleccionado | Revisa que el entorno *FHIR local* esté activo |

## Referencias

- Benson T, Grieve G. *Principles of Health Interoperability: FHIR, HL7 and SNOMED CT*. 4.ª ed. Springer; 2021, caps. 5–7 y 10.
- HL7 International. FHIR R4 · RESTful API: https://hl7.org/fhir/R4/http.html
- HAPI FHIR JPA Server Starter: https://github.com/hapifhir/hapi-fhir-jpaserver-starter
