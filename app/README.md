# arVault - Servicio de cambio de monedas :money_with_wings: :currency_exchange:

La idea es tener un servicio que permita comprar y vender monedas dentro de la wallet. Se usan cuentas internas manejadas por la empresa. Por ahora, no se pueden configurar límites, el único límite es que una cuenta se quede sin plata.

Este servicio **no maneja seguridad**, eso lo resuelve vaultSec, para cuando llega al nginx, el request está autenticado y autorizado. TODO: replicar vaultSec!!! :fearful:.

## Configuración

El servicio tiene un Dockerfile para poder armar una imagen de Docker y levantarlo.

### Almacenamiento

El storage de cuentas, tasas y el log se mantiene, por ahora, en unos archivos JSON. Tienen que existir 3 archivos en el directorio `./state`:

`accounts.json`

Tiene un array con las cuentas de la empresa, con la moneda y el saldo actual. Ejemplo de una cuenta:

    {
        "id": 1,
        "currency": "ARS",
        "balance": 2000000
    }

`rates.json`

Tiene un objeto con las tasas de cambio. Ejemplo de una tasa:

    "ARS": {
        "BRL": 0.00553,
        "EUR": 0.00091,
        "USD": 0.00094
    }

`log.json`

Tiene un array con el log de transacciones del sistema. Ejemplo de una entrada de log:

    {
        "id": "Uml8yqzZ4Mjgk2tKuN6mL",
        "ts": "2025-02-10T00:10:25.202Z",
        "ok": true,
        "request": {
        "baseCurrency": "USD",
        "counterCurrency": "ARS",
        "baseAmount": 100,
        "baseAccountId": 11,
        "counterAccountId": 10
        },
        "exchangeRate": 1064,
        "counterAmount": 106400,
        "obs": null
    }

## Endpoints

### Tasas de cambio

`GET /rates`

Devuelve las tasas de cambio vigentes.

`PUT /rates`

Permite alterar la tasa entre dos monedas. Calcula la recíproca.

    {
    "baseCurrency": "USD",
    "counterCurrency": "ARS",
    "rate": 1064
    }

- `baseCurrency`: Moneda de origen
- `counterCurrency`: Moneda de destino
- `rate`: Tasa de cambio de la moneda origen hacia la destino. La recíproca se calcula como 1/tasa. Notar que no ganamos plata con la operación de cambio.

TODO

- Manejar distintos valores para ganar plata cuando tengamos una buena base de usuarios :smiling_imp:
- Soportar tiers de usuarios para que, si pagan algo por mes, tengan mejor tasa :rocket:

### Cuentas

`GET /accounts`

Devuelve las cuentas internas, sirve para chequear saldos.

`PUT /accounts/<id>/balance`

Actualiza el saldo de una cuenta. No me gusta cómo está hecho esto, otro servicio debería ocuparse de esto (el que hace las transferencias?)

### Cambio

`POST /exchange`

Ejecuta una operación de cambio de monedas

    {
        "baseCurrency": "USD",
        "counterCurrency": "ARS",
        "baseAmount": 100.0,
        "baseAccountId": 11,
        "counterAccountId": 10
    }

- `baseCurrency`: Moneda origen de la transacción
- `counterCurrency`: Moneda destino de la transacción
- `baseAmount`: Importe en moneda origen a cambiar
- `baseAccountId`: ID de la cuenta origen para la operación de cambio (cuenta del cliente)
- `counterAccountId`: ID de la cuenta destino para la operación de cambio (cuenta del cliente)

Este endpoint busca en las cuentas propias las que correspondan a las monedas. Se valida que haya saldo suficiente para efectuar la operación **en la cuenta propia**. **No** se valida que haya saldo en la cuenta del cliente, se espera que lo haga la UI y que no permita la operación.

Todas las operaciones se registran en un log. Ver más abajo.

### Logs

`GET /logs`

Devuelve el log de operaciones. Este log se persiste cada 5 segundos.

## TODO

- No me gusta guardar todo en archivos .json, por ahora va, pero tendría que hacer algo distinto.
- No valida casi nada, solo que los parámetros de los JSON tengan algún valor :collision:
- Ver el tema del manejo de las cuentas, debería ser responsabilidad de otro servicio.
