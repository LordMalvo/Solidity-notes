# Solidity-notes

## Indice
### 1. **[Providers, signers, ABIs y Approval Flows](##providers,-signers,-ABIs-y-Approval-Flows)**
* **[Providers y signers](###providers-y-signers)**
* **[BigNumbers](###BigNumbers)**
* **[ABI](###ABI)**
* **[ERC20 Approval flow](###erc20-approval-flow)**
## Providers, signers, ABIs y Approval Flows
### Providers y signers
En Ethereum, para escribir o leer datos en la blockchain es necesario comunicarse a través de un nodo de Ethereum. Cada nodo tendra un estado dentro de la blockchain, que nos permitirá leer datos, mientras que si queremos escribirlos, será necesario el envio de transacciones.

Un `provider`, es una conexión a un nodo de Ethereum que nos permite leer datos. Se usarán cuando llamemos a funciones que solo lean datos, obtengan el balance de alguna cuenta, obtengan información sobre alguna transacción, etc.

Un `signer`, es una conexión a un nodo de Ethereum que nos permite escribir datos en la blockchain. Se usarán cuando queramos llamar a funciones que escriban datos, transferir ETH entre cuentas, etc. Para realizar todas este tipo de acciones el `signer` necesita acceder a una `private key` que usará para realizar transacciones.

|              | Provider | Signer |
| ------------ | :------: | :----: |
| **Leer**     |    X     |   X    |
| **Escribir** |          |   X    |

Por ejemplo, Metamask, inyecta un `provider` en nuestro navegador, de manera, que otras aplicaciones puedan utilizar ese mismo `provider` para leer datos de la blockchain donde nuestra billetera este conectada. Como Metamask no puede ir por ahi compartiendo nuestra clave privada, lo que hace es permitir a las dApps obtener un `signer`. De esta manera cuando queremos enviar una transaccion con metamask a través de la blockchain, la ventana de Metamask salta preguntando al usuario si quiere confirmar esa transacción.

### BigNumbers
Cuando programamos con solidity, estamos muy acostumbrados a utilizar `uint256`, que tienen un rango que va desde `0` hasta `(2^256) - 1`, lo cual es un rango enorme. De normal se construyen interfaces con Javascript. El problema esque el tipo de dato `number` de Javascript tiene un limite mucho mas bajo que `uint256`.

Imaginemos que Javascript llama a una función de un smart contract que devuelve un `uint256` muchísimo mas grande que lo que soporta un numero en Javascript, bien, simplemente, no lo soporta. Para ello, existe un tipo especial denominado `BigNumber`, y librerias como `ethers.js` y `web3.js`, ya tienen integrado soporte para `BigNumber`.

`BigNumber` es una libreria para Javascript que implementa funciones matemáticas como `add`, `sub`, `mul`, `div`, etc.

### ABI
Las siglas ABI provienen de **A**plication **B**inary **I**nterface.

Cuando se compila código de Solidity, lo hace bajo el bytecode, que a final de cuentas es binario. Este no contiene ni que funciones existen en el contrato, ni que parametros contiene, ni que valores devuelven. No obstante, si quieres llamar a una funcion de Solidity desde dentro de una aplicacion web, necesitas alguna manera de llamar al bytecode correcto, es decir, necesitas convertir el nombre de las funciones y parametros a bytecode y viceversa. 
La `ABI` nos permite precisamente conseguir esto. Cuando se compila un programa de Solidity, automaticamente se genera un fichero `ABI`. Este contiene metadata sobre funciones presentes en el contrato. 

Asi, cuando se quiera interactuar con un contrato, se necesitará tanto su dirección como su `ABI`. `ethers.js` por ejemplo, ya implementa esto, de manera que cuando nos comunicamos con un contrato, automaticamente codifica y decodifica las funciones a bytecode para poder comunicarnos con un nodo de Ethereum. 

A continuación se muestra un ejemplo de como se veria una funcion dentro de la `ABI`:

**Código dentro del fichero de Solidity**
```Solidity
function suma(uint num1, uint num2) public pure returns(uint) {
        return num1 + num2;
}
```

### ERC20 Approval flow
Si queremos pagar o aceptar pagos de tokens ERC20, no es tan simple como utilizar una función 