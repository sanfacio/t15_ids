# T15 — Laboratorio: Despliegue y mantenimiento de software con GitHub Actions

## Del código a producción mediante Git, Pull Requests y CI/CD

**Duración:** 60 minutos aproximadamente  
**Modalidad:** Equipos de 3–4 integrantes  
**Nivel:** Intermedio

## 1. Objetivo

En esta actividad aprenderás a utilizar un flujo básico de integración continua para comprender de manera práctica:

- Gestión de versiones.
- Repositorios Git.
- Ramas y commits.
- Pull Requests.
- Integración continua (CI).
- Pruebas automatizadas.
- Despliegue de software.
- Mantenimiento correctivo.
- Rollback.
- Estrategias de despliegue.
- Automatización mediante GitHub Actions.

El flujo que practicarás será:

```text
Código → Git → Branch → Pull Request → GitHub Actions
      → Pruebas → PASS/FAIL → Corrección → Nueva versión
      → Despliegue → Mantenimiento
```

## 2. Escenario

Imagina que trabajas para una empresa que desarrolla una plataforma de gestión logística.

El equipo necesita controlar versiones, trabajar con nuevas funcionalidades, detectar errores automáticamente y poder regresar a una versión funcional cuando sea necesario.

Construirás una pequeña API llamada **Logistics API** y configurarás un pipeline de CI con GitHub Actions.

## 3. Requisitos

Necesitas:

- Computadora con Internet.
- Cuenta de GitHub.
- Git.
- Node.js.
- npm.
- Visual Studio Code u otro editor.

Se recomienda Windows 11 + WSL Ubuntu, Linux o macOS.

## 4. Verificar herramientas

En una terminal ejecuta:

```bash
git --version
node --version
npm --version
```

Si usas Windows + WSL, puedes entrar con:

```bash
wsl
```

Debes obtener versiones válidas de las tres herramientas.

## 5. Configurar Git

Si es la primera vez que usas Git:

```bash
git config --global user.name "Tu Nombre"
git config --global user.email "tu-correo@example.com"
git config --global --list
```

## 6. Crear el proyecto

```bash
mkdir -p proyectos/logistics-api
cd proyectos/logistics-api
code .
```

Inicializa Node.js:

```bash
npm init -y
```

Instala Express:

```bash
npm install express
```

Instala Jest y Supertest:

```bash
npm install --save-dev jest supertest
```

## 7. Crear `app.js`

Crea `app.js`:

```javascript
const express = require("express");

const app = express();

app.get("/", (req, res) => {
    res.json({
        application: "Logistics API",
        version: "1.0.0",
        status: "running"
    });
});

module.exports = app;
```

## 8. Crear `server.js`

Crea `server.js`:

```javascript
const app = require("./app");

const PORT = process.env.PORT || 3000;

app.listen(PORT, () => {
    console.log(`Server running on port ${PORT}`);
});
```

## 9. Configurar `package.json`

En `package.json`, sustituye la sección `scripts` por:

```json
"scripts": {
    "start": "node server.js",
    "test": "jest"
}
```

## 10. Probar la aplicación

Ejecuta:

```bash
npm start
```

Debes ver:

```text
Server running on port 3000
```

Abre:

```text
http://localhost:3000
```

Debes recibir:

```json
{
  "application": "Logistics API",
  "version": "1.0.0",
  "status": "running"
}
```

Detén el servidor con:

```text
Ctrl + C
```

## 11. Crear una prueba automática

Crea la carpeta `test` y dentro `app.test.js`:

```javascript
const request = require("supertest");
const app = require("../app");

describe("Logistics API", () => {

    test("GET / debe responder correctamente", async () => {
        const response = await request(app).get("/");

        expect(response.statusCode).toBe(200);
        expect(response.body.application).toBe("Logistics API");
        expect(response.body.status).toBe("running");
    });

});
```

Ejecuta:

```bash
npm test
```

El resultado esperado contiene:

```text
PASS
```

## 12. Inicializar Git

```bash
git init
git status
```

Crea `.gitignore`:

```text
node_modules/
.env
coverage/
```

Agrega y confirma:

```bash
git add .
git commit -m "feat: initial logistics API"
git log --oneline
```

## 13. Crear el repositorio en GitHub

Crea en GitHub un repositorio llamado:

```text
logistics-api
```

No agregues otro README, `.gitignore` ni licencia si ya los tienes localmente.

Conecta el repositorio:

```bash
git branch -M main
git remote add origin https://github.com/TU-USUARIO/logistics-api.git
git remote -v
git push -u origin main
```

Reemplaza `TU-USUARIO` por tu usuario real de GitHub.

## 14. Crear una rama para una nueva funcionalidad

No desarrollaremos directamente sobre `main`.

```bash
git checkout -b feature/health-check
git branch
```

Debes ver:

```text
* feature/health-check
  main
```

## 15. Agregar `GET /health`

Modifica `app.js` agregando:

```javascript
app.get("/health", (req, res) => {
    res.json({
        status: "OK"
    });
});
```

Agrega la prueba en `test/app.test.js`:

```javascript
test("GET /health debe responder OK", async () => {
    const response = await request(app).get("/health");

    expect(response.statusCode).toBe(200);
    expect(response.body.status).toBe("OK");
});
```

Ejecuta:

```bash
npm test
```

Debe aparecer:

```text
PASS
```

## 16. Crear commit y subir la rama

```bash
git add .
git commit -m "feat: add health check"
git push -u origin feature/health-check
```

## 17. Crear un Pull Request

En GitHub selecciona **Compare & pull request**.

Configura:

```text
base: main
compare: feature/health-check
```

Título:

```text
Add health check endpoint
```

Descripción:

```text
This Pull Request adds a health check endpoint
to the Logistics API and includes automated tests.
```

Crea el Pull Request.

### ¿Por qué usamos un Pull Request?

Permite revisar un cambio antes de incorporarlo a la rama principal:

```text
main
  ↑
Pull Request
  ↑
feature/health-check
```

## 18. Crear el pipeline CI

Crea:

```text
.github/workflows/ci.yml
```

Contenido:

```yaml
name: CI Logistics API

on:
  push:
    branches:
      - main
      - "feature/**"

  pull_request:
    branches:
      - main

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20

      - name: Install dependencies
        run: npm ci

      - name: Run tests
        run: npm test
```

Este pipeline realiza:

```text
Checkout
   ↓
Setup Node.js
   ↓
npm ci
   ↓
npm test
   ↓
PASS / FAIL
```

## 19. Subir el pipeline

```bash
git add .
git commit -m "ci: add GitHub Actions pipeline"
git push
```

En GitHub abre:

**Actions → CI Logistics API**

La ejecución debe mostrar pasos exitosos:

```text
Checkout repository       ✓
Setup Node.js             ✓
Install dependencies      ✓
Run tests                 ✓
```

## 20. Simular un error

En `app.js`, cambia:

```javascript
status: "OK"
```

por:

```javascript
status: "ERROR"
```

Ejecuta:

```bash
npm test
```

Ahora debe aparecer:

```text
FAIL
```

Sube el cambio:

```bash
git add .
git commit -m "fix: simulate health check failure"
git push
```

Regresa a **GitHub → Actions**.

Debes observar una ejecución fallida.

### Pregunta

¿Por qué es importante detectar este error automáticamente antes de desplegar?

Considera:

- Detección temprana.
- Automatización.
- Reducción de errores.
- Protección de la versión principal.
- Confianza en los cambios.

## 21. Corregir el error

Regresa `app.js` a:

```javascript
status: "OK"
```

Ejecuta:

```bash
npm test
```

Debe aparecer:

```text
PASS
```

Sube la corrección:

```bash
git add .
git commit -m "fix: restore health check status"
git push
```

Comprueba nuevamente GitHub Actions.

## 22. Rollback con `git revert`

Consulta el historial:

```bash
git log --oneline
```

`git revert` crea un nuevo commit que deshace los cambios de otro commit.

Conceptualmente:

```text
Versión 1
   ↓
Versión 2
   ↓
Versión 3
   ↓
Revert
   ↓
Nueva versión que deshace un cambio
```

Para revertir un commit específico:

```bash
git revert ID_DEL_COMMIT
```

Después:

```bash
git log --oneline
git push
```

Verifica nuevamente GitHub Actions.

> Identifica cuidadosamente el commit antes de ejecutar `git revert`.

## 23. Clasificar tipos de mantenimiento

Clasifica cada situación como **Correctivo, Adaptativo, Perfectivo o Preventivo**.

### Caso 1

La aplicación tiene un error que provoca que se cierre inesperadamente.

**Respuesta:** ____________________

### Caso 2

La aplicación debe modificarse porque una nueva versión de un sistema operativo cambió una API.

**Respuesta:** ____________________

### Caso 3

El cliente solicita una nueva funcionalidad para generar reportes.

**Respuesta:** ____________________

### Caso 4

El equipo refactoriza código para reducir la posibilidad de futuros problemas.

**Respuesta:** ____________________


## 24. Analizar estrategias de despliegue

### A. Despliegue básico

```text
Versión 1
   ↓
Actualizar
   ↓
Versión 2
```

Es sencillo, pero puede generar interrupciones o riesgos durante la actualización.

### B. Blue-Green

```text
             ┌── Blue ── Versión 1
Usuarios ────┤
             └── Green ─ Versión 2
```

La nueva versión se prepara y prueba en Green antes de cambiar el tráfico.

### C. Canary

```text
Usuarios
   │
   ├── 95% → Versión 1
   │
   └── 5%  → Versión 2
```

Una pequeña parte de los usuarios recibe inicialmente la nueva versión.

### D. Continuous Deployment

```text
Commit
   ↓
Build
   ↓
Test
   ↓
Deploy
   ↓
Monitor
```

Los cambios pueden avanzar automáticamente si cumplen las condiciones establecidas.

## 25. Reto de decisión

Supón que la empresa tiene miles de usuarios y quiere minimizar el riesgo de una nueva versión.

Selecciona:

```text
[ ] Básica
[ ] Blue-Green
[ ] Canary
[ ] Continuous Deployment
```

Responde:

1. ¿Por qué elegiste esta estrategia?
2. ¿Cuál es el principal riesgo?
3. ¿Cómo realizarías un rollback?
4. ¿Qué pruebas realizarías antes del despliegue?
5. ¿Qué métricas revisarías después del despliegue?

## 26. Flujo completo

Debes poder explicar este proceso:

```text
                 DESARROLLO
                     │
                     ▼
                  Git
                     │
                     ▼
                  Branch
                     │
                     ▼
              Pull Request
                     │
                     ▼
              GitHub Actions
                     │
                     ▼
             Automated Tests
                │         │
              FAIL       PASS
                │         │
                ▼         ▼
             Fix       Continue
                          │
                          ▼
                       Deploy
                          │
                          ▼
                     Production
                          │
                          ▼
                      Monitor
                          │
                          ▼
                    Maintenance
                          │
                          ▼
                    New Version
                          │
                          └──────► Git
```

## 27. Preguntas de reflexión

Responde individualmente:

1. ¿Cuál es la función de Git dentro del proceso de mantenimiento?
2. ¿Cuál es la diferencia entre una rama y `main`?
3. ¿Qué problema ayuda a resolver un Pull Request?
4. ¿Qué ventaja tiene ejecutar pruebas automáticamente?
5. ¿Qué sucede cuando GitHub Actions detecta una prueba fallida?
6. ¿Por qué `git revert` puede ser útil durante el mantenimiento?
7. ¿Cuál es la diferencia entre mantenimiento correctivo y perfectivo?
8. ¿Por qué el despliegue no debe considerarse el final del ciclo de vida del software?

## 28. Evidencias de entrega

Debes entregar:

### Evidencia 1 — Repositorio

URL:

```text
_______________________________________________
```

Debe contener:

```text
app.js
server.js
package.json
package-lock.json
test/app.test.js
.github/workflows/ci.yml
.gitignore
```

### Evidencia 2 — Historial Git

Captura de:

```bash
git log --oneline
```

### Evidencia 3 — Pull Request

Captura del Pull Request de:

```text
feature/health-check
```

hacia:

```text
main
```

### Evidencia 4 — Pipeline exitoso

Captura de GitHub Actions mostrando `Success`.

### Evidencia 5 — Pipeline fallido

Captura de la ejecución donde se detectó el error intencional.

### Evidencia 6 — Recuperación

Captura de una ejecución posterior exitosa.

### Evidencia 7 — Estrategia de despliegue

Documento o sección con:

- Estrategia seleccionada.
- Justificación.
- Riesgos.
- Rollback.
- Pruebas.
- Monitoreo.

## 29. Checklist final

- [ ] La aplicación funciona localmente.
- [ ] `npm test` funciona.
- [ ] Existe un repositorio GitHub.
- [ ] Existe una rama adicional.
- [ ] Creé un Pull Request.
- [ ] Existe un workflow de GitHub Actions.
- [ ] El pipeline ejecutó correctamente.
- [ ] Provocamos un error intencional.
- [ ] Observamos un pipeline fallido.
- [ ] Corregimos el problema.
- [ ] Observamos un pipeline exitoso.
- [ ] Revisamos el historial Git.
- [ ] Comprendemos `git revert`.
- [ ] Clasificamos los tipos de mantenimiento.
- [ ] Analizamos estrategias de despliegue.
- [ ] Contestamos las preguntas de reflexión.
- [ ] Tenemos todas las evidencias.

# 30. Reto avanzado — Docker

Esta sección es opcional.

Crea `Dockerfile`:

```dockerfile
FROM node:20-alpine

WORKDIR /app

COPY package*.json ./

RUN npm ci --omit=dev

COPY . .

EXPOSE 3000

CMD ["node", "server.js"]
```

Construye la imagen:

```bash
docker build -t logistics-api:1.0 .
```

Verifica:

```bash
docker images
```

Ejecuta:

```bash
docker run -p 3000:3000 logistics-api:1.0
```

Prueba:

```text
http://localhost:3000
```

y:

```text
http://localhost:3000/health
```

El flujo avanzado que debes imaginar es:

```text
Git Push
   ↓
Tests
   ↓
Docker Build
   ↓
Docker Image
   ↓
Container Registry
   ↓
Deployment
   ↓
Monitoring
```

No es necesario implementar esta parte para completar el laboratorio.

# 31. Solución de problemas

## `git: command not found`

En Ubuntu/WSL:

```bash
sudo apt update
sudo apt install git
```

Después:

```bash
git --version
```

## `node: command not found`

Node.js no está instalado o no está disponible en el PATH. Verifica:

```bash
node --version
```

## `npm: command not found`

Verifica la instalación de Node.js:

```bash
node --version
npm --version
```

## Puerto 3000 ocupado

Puedes utilizar otro puerto:

```bash
PORT=3001 npm start
```

Después abre:

```text
http://localhost:3001
```

## `npm test` falla

Ejecuta:

```bash
npm install
npm test
```

Revisa:

```text
app.js
test/app.test.js
package.json
```

## GitHub Actions falla en `npm ci`

Asegúrate de haber subido:

```text
package.json
package-lock.json
```

Puedes comprobarlo con:

```bash
git status
```

Si falta `package-lock.json`:

```bash
git add package-lock.json
git commit -m "chore: add package lock"
git push
```

## GitHub solicita autenticación

GitHub no utiliza normalmente la contraseña de la cuenta para autenticación Git mediante HTTPS. Utiliza el mecanismo de autenticación que GitHub indique, como GitHub CLI, Credential Manager o un Personal Access Token cuando corresponda.

# 32. Conceptos clave

Al terminar debes poder explicar con tus propias palabras:

**Git:** sistema de control de versiones.

**Commit:** registro de un conjunto de cambios.

**Branch:** línea independiente de desarrollo.

**Pull Request:** solicitud para incorporar cambios de una rama a otra.

**Continuous Integration:** integración y validación automatizada de cambios.

**Pipeline:** secuencia automatizada de tareas.

**Automated Test:** prueba ejecutada automáticamente.

**Deployment:** proceso de poner una versión del software disponible en un ambiente.

**Rollback:** regreso controlado a una versión o estado anterior.

**Corrective Maintenance:** corrección de errores.

**Adaptive Maintenance:** adaptación a cambios del entorno.

**Perfective Maintenance:** mejora o ampliación de funcionalidades.

**Preventive Maintenance:** acciones para reducir problemas futuros.

# 33. Conclusión

El objetivo de esta práctica no es solamente aprender comandos de Git o crear un workflow.

Debes comprender cómo las prácticas se integran:

```text
Gestión de versiones
        ↓
       Git
        ↓
     GitHub
        ↓
   Pull Request
        ↓
        CI
        ↓
Pruebas automatizadas
        ↓
     Deploy
        ↓
   Producción
        ↓
    Monitoreo
        ↓
   Mantenimiento
        ↓
   Nueva versión
        ↓
       Git
```

La idea fundamental es:

> **El despliegue de software no representa el final del trabajo. Después del despliegue comienza un proceso continuo de monitoreo, mantenimiento, corrección y evolución del software.**

## Entrega final

Antes de entregar proporciona:

1. URL del repositorio GitHub.
2. Captura del Pull Request.
3. Captura de un pipeline exitoso.
4. Captura de un pipeline fallido.
5. Captura de la recuperación.
6. Historial de commits.
7. Respuestas a las preguntas de reflexión.
8. Análisis de la estrategia de despliegue seleccionada.

**¡Laboratorio terminado!**
