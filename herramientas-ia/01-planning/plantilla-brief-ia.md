# Tecnical brief master template

## 1. Titulo de la tarea

Ejemplo:
Servicio desacoplado de cálculo de impuestos.

---

## 2. Contexto
Describe un resumen del problema actual, los objetivos y desafíos del projecto.

Incluye:

- [ ] Factores de éxito
- [ ] Como funciona el sistema actualmente.
- [ ] Qué problema existe (acoplamiento, deuda técnica, escalabilidad, etc).
- [ ] Cuales son las consecuencias actuales de dicho problema.
- [ ] Qué se busca lograr con esta implementación (objetivo).
- [ ] El plan de ejecución.
- [ ] Metas medibles.
- [ ] Usuarios finales y sus necesidades específicas.
- [ ] Los miembros del proyecto y el conocimiento técnico de cada uno (opcional).
- [ ] Valor de utilizar IA en lugar de la solución actual.


## 3. Requerimientos técnicos
### Lenguaje front y back permitidos con sus versiones a utilizar, guía de patrones, manejo de exceptions y ejemplos async.

- [ ] **Python 3.13+** -- patrones/python-pattern.md
- [ ] **Go 1.24.0+** -- go-patterns.md
- [ ] **React 18.0+** -- react-patterns.md
- [ ] **PostgreSQL 17+**
- [ ] **MySQL 8.0.44+**
- [ ] **MongoDB 7.0+**

Infraestructura estimada a utilizar
Segmentar de manera clara los servicios a crear.

### Arquitectura

## Definir patrones de diseño a utilizar para la implentación

### Matriz de decisiones rápidas

| Caso | Go | Python | React/Next.js |
|----------|-----|-----------|---------------|
| **REST API** | Hexagonal + interfaces | FastAPI service layer | — |
| **Agent/LLM** | — | Langchain + FastAPI | — |
| **Data access** | Repository pattern | Repository + SQLAlchemy | — |
| **Async work** | Goroutines | asyncio | — |
| **UI** | — | — | TanStack Query + Zustand |
| **Microservice** | All: event-driven, resilience patterns, observability |

### Establecer reglas/requerimientos ténicos específicos:
- [ ] Identifica las fuentes de datos, calidad y volumen.
- [ ] Idioma de lenguajes primero: usar patrones nativos al lenguaje, no tomar patrones de otros y querer adaptarlos.
- [ ] Simplicidad por sobre magia: priorizar código que sea investigable/debuggeable por sobre abstracciones inteligentes.
- [ ] Establecer las mejores prácticas en los lenguajes a utilizar.
- [ ] Modelo de IA: ML supervisado, IA Generativa (NLP Transformers)
- [ ] Testability: utilizar interfaces limpias/prolijas con componentes testeables y de bajo acoplamiento.
- [ ] Observabilidad: logging estructurado, métricas y trazabilidad del código desde el inicio del proyecto.
- [ ] Resiliencia: retries y timeout para sistemas distribuidos. 
- [ ] Especificar si se necesita mayor escalabilidad o refactoring, que sean soluciones mantenibles y confiables.
- [ ] Progressive disclosure: Muestra lo necesario y oculta la complejidad.
- [ ] Consistent patterns: Usa componentes de diseño de sistema establecidos. 

### Mostrar la estructura del directorio
- Ejemplo para FastAPI:

/project-root
├── frontend/
│   ├── public/
│   ├── src/
│   │   ├── components/
│   │   ├── pages/
│   │   ├── services/ or api/
│   │   ├── App.js
│   │   └── index.js
│   ├── package.json
│   └── ... (other React files/folders)
├── backend/
│   ├── app/
│   │   ├── api/ or routers/
│   │   ├── core/
│   │   ├── db/
│   │   ├── models/
│   │   ├── schemas/
│   │   ├── main.py
│   │   └── __init__.py
│   ├── venv/ or .venv/
│   ├── requirements.txt or pyproject.toml
│   └── ... (other backend files/folders)
├── .gitignore
└── README.md



## 4. Constraints
- [ ] Prioridades del cliente y preocupaciones
- [ ] Deadline del proyecto
- [ ] Evitar contenido muy genérico.
- [ ] Presupuesto
- [ ] Normativas de privacidad.
- [ ] Tecnologia obligatoria.
- [ ] Librerías externas no permitidas.


## 5. Definition of Done

- [ ]Criterios para considerar el brief como completado.
- [ ]Tests que deben dar success.
- [ ]Metricas de performance logradas.

