1. Visión General del ProyectoEl objetivo es construir un sistema de gestión de inventario web para una tienda de juguetes. El sistema debe ser ágil, visualmente atractivo (UI/UX temática de juguetes) y robusto, garantizando integridad de datos bajo concurrencia (ej. múltiples administradores o compras simultáneas).Stack Tecnológico:Backend: Python (Flask o FastAPI).Base de Datos: PostgreSQL.ORM / Conector: SQLAlchemy o psycopg2 / psycopg3.Frontend: HTML5, CSS3 (CSS Vanilla modular o Tailwind CSS), JavaScript (vanilla para agilidad e interactividad).Testing: pytest (Backend), unittest/coverage.2. Directrices de UX/UILa interfaz debe reflejar el dinamismo y color de una tienda de juguetes sin perder profesionalismo operativo.Estética: Colores vibrantes pero balanceados (paleta accesible), bordes suaves, micro-interacciones (hover en productos, badges de stock).Usabilidad:Búsqueda en tiempo real e incremental de juguetes (por categoría, edad recomendada, marca, rango de precio).Alertas visuales para Stock Bajo (ej. $< 5$ unidades) y Agotado.Feedback inmediato mediante notificaciones Toast (éxito, error, advertencias).Formularios ágiles con validación en cliente (HTML5/JS) y backend.3. Base de Datos, Transacciones y Concurrencia (PostgreSQL)Requerimientos de Base de Datos:Modelos clave: Products, Categories, StockMovements, Suppliers, Users/Roles.Integridad Referencial: Foreign Keys, Constraints CHECK (ej. stock >= 0, price > 0).Manejo de Concurrencia y Transacciones:Transacciones ACID: Todas las operaciones de actualización o reducción de inventario deben encapsularse en bloques de transacción (BEGIN, COMMIT, ROLLBACK).Bloqueo Pesimista / Optimista:Usar SELECT ... FOR UPDATE en operaciones críticas (ej. descuento masivo de stock) para evitar Race Conditions / Lost Updates.O manejar versión de fila (Optimistic Locking) mediante una columna version o updated_at.Nivel de Aislamiento: Configurar por defecto en READ COMMITTED o REPEATABLE READ según el flujo de movimiento de almacén.4. Reglas y Estándares de Código (Python & DB)Python:Documentación: Todo módulo, clase y función debe incluir Docstrings en formato Google o Docstring Estándar PEP 257.Type Hints: Obligatorio usar anotaciones de tipo (typing).Manejo de Excepciones:Captura explícita de errores de base de datos (psycopg.errors.IntegrityError, OperationalError, DatabaseError).No usar except Exception: genéricos sin logging adecuado.Respuestas HTTP estructuradas con códigos de estado correctos (400, 404, 409 Conflict, 500).Ejemplo de Estructura de Operación Backend:def update_product_stock(db_session, product_id: int, quantity_change: int) -> Product:
    """
    Actualiza el stock de un juguete garantizando el bloqueo de fila.

    Args:
        db_session: Sesión activa de base de datos.
        product_id (int): ID único del producto.
        quantity_change (int): Cantidad a sumar o restar del stock.

    Returns:
        Product: Instancia del producto actualizado.

    Raises:
        ValueError: Si el stock resultante es menor a cero.
        ProductNotFoundError: Si el producto no existe.
        DatabaseError: En caso de falla en la transacción.
    """
    try:
        # Bloqueo de fila para concurrencia segura
        product = db_session.query(Product).with_for_update().filter_by(id=product_id).first()
        
        if not product:
            raise ProductNotFoundError(f"Producto con ID {product_id} no encontrado.")
        
        new_stock = product.stock + quantity_change
        if new_stock < 0:
            raise ValueError(f"Stock insuficiente para {product.name}. Disponible: {product.stock}")
        
        product.stock = new_stock
        db_session.commit()
        return product
    except Exception as e:
        db_session.rollback()
        logger.error(f"Error al actualizar stock para producto {product_id}: {str(e)}")
        raise
5. Estrategia de Pruebas (Testing)Todo código propuesto o generado debe incluir pruebas asociadas:Pruebas Unitarias (tests/unit/):Validaciones de modelo.Lógica de negocio (cálculo de precios, alertas de stock).Pruebas de Integración (tests/integration/):Enpoints de API REST / Rutas HTML.Transacciones de base de datos usando base de datos de pruebas PostgreSQL en Docker.Pruebas de Concurrencia (tests/concurrency/):Simulación de solicitudes simultáneas (utilizando threading o asyncio) para verificar la prevención de race conditions en la actualización de stock.6. Estructura del Proyectotoystore_inventory/
├── app/
│   ├── static/
│   │   ├── css/
│   │   │   └── styles.css
│   │   └── js/
│   │       └── inventory.js
│   ├── templates/
│   │   ├── base.html
│   │   ├── inventory_list.html
│   │   └── product_form.html
│   ├── controllers/
│   ├── models/
│   ├── services/
│   └── database.py
├── tests/
│   ├── test_services.py
│   └── test_concurrency.py
├── docs/
│   └── schema.sql
├── AGENTS.md
├── requirements.txt
└── README.md
7. Instrucciones para los AgentesAl implementar o modificar funcionalidades en este proyecto, asegúrate de:Validar siempre las entradas en el frontend y backend.Incluir pruebas unitarias/integración para cualquier función nueva.Asegurar que las consultas SQL o llamadas al ORM estén libres de vulnerabilidades (Inyección SQL) y gestionen la concurrencia.Documentar las APIs y actualizar este archivo si se introducen nuevos estándares.