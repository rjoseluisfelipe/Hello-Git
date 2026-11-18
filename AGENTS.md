# AGENTS.md

## Estado real del repositorio
- Greenfield: no hay código, README, `requirements.txt`, manifiestos, CI ni tests. Este archivo es el único contenido verificable.
- Están vacíos `code/`, `spec/constitution/` y `spec/features/001_tarea/`. No existen `app/`, `tests/`, `docs/`, `README.md` ni `requirements.txt`: el árbol del spec anterior era aspiracional, no real.
- No hay comandos de arranque, test ni lint verificables. No inventarlos; crearlos junto al primer manifiesto o script.

## Decisiones pendientes: confirmar antes de crear estructura
- Framework backend: Flask o FastAPI. Acceso a datos: SQLAlchemy o psycopg2/3. CSS: vanilla modular o Tailwind.
- Ubicación de la aplicación: `code/` existe pero está vacío; no asumir `app/` en la raíz.

## Reglas de dominio y concurrencia (spec original)
- PostgreSQL. Modelos clave: Products, Categories, StockMovements, Suppliers, Users/Roles. FK y CHECK (`stock >= 0`, `price > 0`).
- Toda actualización de stock va en transacción ACID con COMMIT/ROLLBACK y bloqueo de fila (`SELECT ... FOR UPDATE`) o versionado optimista (`version` / `updated_at`). Aislamiento previsto: READ COMMITTED o REPEATABLE READ.
- Stock bajo = menos de 5 unidades; alertas visuales de stock bajo y agotado. Búsqueda incremental por categoría, edad recomendada, marca y rango de precio. Notificaciones Toast; validación en cliente y servidor.

## Estándares de código
- Python con type hints obligatorios y docstrings (Google o PEP 257) en módulos, clases y funciones.
- Capturar explícitamente `psycopg.errors.IntegrityError`, `OperationalError` y `DatabaseError`: rollback y log. Prohibido `except Exception:` genérico sin logging; el ejemplo del spec anterior lo incumplía, no copiarlo literalmente.
- HTTP: 400 validación, 404 no encontrado, 409 conflicto por stock/concurrencia, 500 error interno.
- Consultas parametrizadas vía ORM; nunca interpolación de SQL.

## Pruebas (requeridas para toda funcionalidad nueva)
- Unitarias: validaciones de modelo y lógica de stock/precios.
- Integración: endpoints o rutas y transacciones contra PostgreSQL de prueba en Docker.
- Concurrencia: hilos o asyncio simultáneos para verificar ausencia de lost updates.
- Framework previsto: pytest (también se contempla unittest/coverage).
