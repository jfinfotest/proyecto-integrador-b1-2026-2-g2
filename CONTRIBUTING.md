```markdown
# Guía de Contribución para jfinfotest/proyecto-integrador-b1-2026-2-g2

¡Hola! 👋 Nos alegra mucho que estés aquí y quieras contribuir a nuestro **Proyecto Integrador**. Para mantener la calidad del código, asegurar la estabilidad del sistema y facilitar la colaboración en equipo, hemos definido este manual de procesos y estándares de ingeniería.

Por favor, tómate unos minutos para leer esta guía antes de escribir código o enviar un Pull Request.

---

## 1. Convención de Ramas (Git Branching Strategy)

Trabajaremos utilizando un flujo basado en ramas temáticas (Feature Branch Workflow). Nunca se debe hacer *push* directo a la rama `main` o `develop`.

Las ramas deben seguir estrictamente el siguiente formato:
`<tipo>/<descripcion-corta-en-kebab-case>`

### Tipos de ramas permitidas:
*   `feature/` : Para el desarrollo de nuevas funcionalidades o historias de usuario.
    *   *Ejemplo:* `feature/integrar-spring-security`
*   `bugfix/` : Para la corrección de errores encontrados durante el desarrollo o pruebas.
    *   *Ejemplo:* `bugfix/fix-null-pointer-usuario-service`
*   `hotfix/` : Para correcciones urgentes en producción (rama `main`).
    *   *Ejemplo:* `hotfix/error-conexion-bd-produccion`
*   `refactor/` : Para refactorización de código sin alterar la funcionalidad externa.
    *   *Ejemplo:* `refactor/limpiar-endpoints-facturacion`
*   `test/` : Para la implementación o mejora de pruebas unitarias/de integración.
    *   *Ejemplo:* `test/agregar-unit-tests-auth`

---

## 2. Convención de Commits (Conventional Commits)

Nuestros mensajes de commit deben ser claros, descriptivos y seguir la especificación de **Conventional Commits**. Esto nos permite generar changelogs automáticos y entender el historial del proyecto rápidamente.

**Estructura:**
```text
<tipo>(<alcance>): <descripción breve>

[cuerpo opcional]

[pie de página opcional]
```

### Tipos de commits:
*   `feat`: Una nueva característica o funcionalidad.
*   `fix`: Una corrección de un bug.
*   `docs`: Cambios exclusivos en la documentación (README, CONTRIBUTING, JavaDoc).
*   `style`: Cambios que no afectan el significado del código (espacios en blanco, formato, punto y coma faltantes, etc.).
*   `refactor`: Un cambio en el código que no corrige un error ni añade una característica (mejora de estructura/legibilidad).
*   `test`: Agregar pruebas faltantes o corregir las existentes.
*   `chore`: Cambios en el proceso de construcción, herramientas auxiliares o dependencias (ej. `pom.xml`, plugins de Maven).

### Ejemplos prácticos:
```bash
git commit -m "feat(auth): implementar endpoint de login con JWT"
git commit -m "fix(usuario): solucionar validacion de correo duplicado en registro"
git commit -m "docs(readme): actualizar instrucciones de ejecucion con Docker"
git commit -m "refactor(dto): migrar records de Java para representar respuestas de API"
git commit -m "test(service): agregar pruebas unitarias para UsuarioServiceImpl"
git commit -m "chore(deps): actualizar version de Spring Boot a 3.2.x"
```

---

## 3. Flujo de Trabajo de Pull Requests (PR)

Cuando tu funcionalidad o corrección esté lista, debes abrir un Pull Request hacia la rama `develop` (o la rama base correspondiente).

### Checklist para el PR:
- [ ] Tu código compila correctamente y pasa todas las pruebas locales (`mvn clean verify`).
- [ ] Has añadido pruebas unitarias y/o de integración para el nuevo código.
- [ ] El código sigue los estándares de estilo de Java y Spring Boot definidos en este repositorio.
- [ ] No incluyes archivos innecesarios (como binarios, `.class`, configuraciones locales del IDE, o archivos `application-local.properties` con credenciales).
- [ ] El título de tu PR sigue la misma convención que los commits (ej: `feat(api): agregar controlador de productos`).

### Descripción del PR:
Utiliza la plantilla predeterminada al crear el PR y asegúrate de rellenar:
1. **Qué resuelve:** Breve descripción del problema o historia de usuario abordada.
2. **Cómo se probó:** Pasos realizados para verificar el funcionamiento localmente.
3. **Screenshots/Logs:** Si aplica (ej. respuestas de Postman o trazas de consola).

---

## 4. Estándares de Código y Buenas Prácticas (Java & Spring Boot)

Dado que nuestro stack es **Java** con **Spring Boot**, adherimos a las siguientes normas de ingeniería:

1. **Estilo de código:** 
   * Utiliza la convención de nombres estándar de Java (CamelCase para clases y métodos, camelCase para variables, UPPER_CASE para constantes).
   * Evita "números mágicos" o cadenas de texto hardcodeadas; utiliza constantes (`static final`) o archivos de propiedades.
2. **Arquitectura:**
   * Respeta la separación de capas: `Controller` -> `Service` -> `Repository`.
   * Los controladores (`@RestController`) deben ser limpios y delegar toda la lógica de negocio a la capa de servicios.
   * Utiliza inyección de dependencias por constructor (evita el uso de `@Autowired` en campos/fields).
3. **Manejo de Errores:**
   * Utiliza excepciones personalizadas y un manejador global de excepciones mediante `@ControllerAdvice` / `@RestControllerAdvice` para retornar respuestas HTTP estandarizadas.
4. **Clean Code:**
   * Escribe código auto-documentado. Añade JavaDoc únicamente en interfaces públicas o lógica de negocio compleja.
   * Limita la complejidad ciclomática de los métodos. Si un método es muy largo, divídelo en métodos privados más pequeños.

---

¡Gracias por contribuir a que este proyecto sea un éxito! Si tienes dudas, abre un *Issue* o consulta con el Líder Técnico del equipo. 🚀
```