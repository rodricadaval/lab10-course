# Plantilla de revisión

## 1. Alucionaciones de librerías

### Pregunta clave

El import realmente existe ?

**Verificar:**

- [ ]Documentación oficial
- [ ] Existencia real de paquetes
- [ ] Compatibilidad con la versión usada

### Checklist

- [ ] los imports existen
- [ ] las funciones usadas son reales
- [ ] la API corresponde a la version actual

## 2. Logica del negocio

### Pregunta clave

- La implementación hace match con el diseño y plan original ?

**Errores comunes generados por la IA**

- [ ] Que haya scopes no planificados
- [ ] No se cumple el criterio de aceptación
- [ ] Existen cambios que entran en conflicto con el features existentes.
- [ ] Los Pull Requests son simples y poco detallados.
- [ ] Calculos incorrectos
- [ ] Redondeos incorrectos
- [ ] manejo incorrecto de fechas
- [ ] uso de float para dinero

### Checklist

- [ ] scopes planificados
- [ ] los calculos son correctos
- [ ] el manejo de fechas es consistente
- [ ] no se usa float para dinero
- [ ] edge cases están contemplados

### Calidad del código

#### Naming Conventions

- [ ] Las variables tienen nombres claros y descriptivos
- [ ] Las funciones/métodos describen lo que hacen (convención verbo_sustantivo)
- [ ] Los nombres de las clases son sustantivos que describen una Entidad
- [ ] CONSTANTES tienen formato UPPER_SNAKE_CASE
- [ ] Nombres no ambiguos

#### DRY (Don't Repeat Yourself)

- [ ] No repetir código que puede ser encapsulado en una función
- [ ] Librerias/Utilidades compartidas deben estar en un directorio apropiado
- [ ] Que haya patrones de código del tipo COPY-PASTE
- [ ] Componentes reutilizables están correctamente definidos. 

#### YAGNI (You Aren't Gonna Need It)

- [ ] No sobre complejizar código (overengineering)
- [ ] Variables y librerias sin uso no pueden existir.
- [ ] No pueden haber funcionalidades "por si acaso".
- [ ] En caso de procedimientos complejos, deben ser justificados por los requerimientos.

#### Principios de responsabilidad individual

- [ ] Cada función debe tener un solo propósito y el mismo debe ser claro.
- [ ] Las clases deben tener una sóla razón para cambiar.
- [ ] Las funciones no pueden hacer demasiadas cosas.
- [ ] La separación de contextos se mantiene correctamente.
- [ ] Funciones largas se separan en pequeñas y testeables micro-funciones. 

### Arquitectura

#### Patrones existentes

- [ ] El código sigue los patrones de diseño definidos en brief y plan.
- [ ] La organización de archivos coincide con la estructura de directorios del projecto.
- [ ] Uso de capas de arquitectura consistentes (controller, service, repository)

#### Separación de dominios

- [ ] La lógica de acceso a los datos tiene repositorios/DAOS dedicados.
- [ ] Las configuraciones no están hardcodeadas.
- [ ] El manejo de errores es consistentes con los patrones de diseño definidos.
- [ ] Las dependencias se inyectan correctamente y no están fuertemente acopladas.

#### Patrones de diseño

- [ ] Se utilizan patrones de diseño donde se necesita y no hay sobreuso
- [ ] Factory para creación de objetos
- [ ] Observer para manejo de eventos
- [ ] Strategy para variaciones de algoritmos
- [ ] Prohibidos los Anti-patterns (dependencias circulares, Clases que hacen todo, etc.)

### Testing

#### Coverage

- [ ] Los Unit tests cubren la lógica principal
- [ ] Se testean los Edge cases (nulls, empty collections, boundaries)
- [ ] Se testean los casos de error (exceptions, validation failures)
- [ ] Existen tests de integracion y verifican la interaccion entre componentes
- [ ] Coverage es al menos el 85% para nuevo código

#### Calidad

- Los tests son descriptivos y fáciles de entender.
- Los nombres de los tests describen lo que se pretende y cuál es la salida esperada.
- Los tests son independientes y pueden ejecutarse en cualquier orden
- Que no haya interdependencia de datos inyectados para tests.
- Los Mocks se usan apropiadamente.
- Los tests se aplican sobre el código de la aplicación y no sobre el comportamiento del framework.



## 3. Seguridad

### Input Validation
- [ ] Todos los inputs de usuario están validados
- [ ] Los inputs que vengan de fuentes externas no son confiables
- [ ] Se hacen validaciones de formato y longitud de texto.
- [ ] Utilizar campos del tipo Whitelist siempre que sea posible

### Secrets Management
- [ ] API keys, tokens, y passwords no hardcodeados
- [ ] No commitear secrets en archivos de configuración a git.
- [ ] Datos sensibles deben ser introducidos mediante variables de entorno.
- [ ] No exponer credenciales de acceso a bases de datos en el código ni los logs.

### Protección de inyecciones
- [ ] Parameterized statements (no usar concatenación de strings) para consultas SQL
- [ ] No ejecutar código dinámico (eval, exec)
- [ ] Inyeccion via Command es prevenido

### Authenticación & Autorización
- [ ] Chequeos de autenticación están correctamente establecidos.
- [ ] Se verifica Autorización sobre recursos protegidos.
- [ ] Validación por sesión/token se implementa correctamente.
- [ ] Mejores prácticas de seguridad de passwords (hashing, salting, sha-256 algorithms)
- [ ] No se introducen vulneravilidades del tipo "Escalado de privilegios"

### Protección de datos
- [ ] Los datos sensibles son encriptados a través de los requests (HTTPS/TLS)
- [ ] El manejo de permisos y datos usa la lógica "menores permisos posibles".
- [ ] El acceso a datos es loggeado donde es requerido.
