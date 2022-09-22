# Solidity-notes

## Indice
### 1. **[Implementación ERC20](#implementación-erc20)**
* **[Interfaz IERC20](#interfaz-ierc20)**
### 2. **[Providers, signers, ABIs y Approval Flows](#providers,-signers,-abis-y-approval-flows)**
* **[Providers y signers](#providers-y-signers)**
* **[BigNumbers](#bignumbers)**
* **[ABI](#abi)**
* **[ERC20 Approval flow](#erc20-approval-flow)**

## **1.** Implementación ERC20

### 1.1 Interfaz IERC20
```solidity
function totalSupply() external view returns(uint256);
```
* Función `totalSupply`: Devuelve la cantidad de tokens que existen. 
  
```solidity
function balanceOf(address account) external view returns(uint256);
```
* Función `balanceOf`: Devuelve la cantidad de tokens que posee cierta dirección `account`. 

```solidity
function transfer(address recipient, uint256 amount) external returns(bool);
```
* Función `transfer`: Mueve una cantidad (`amount`) de tokens desde la cuenta que llama a la funcion hacia el `recipient`. Devuelve un valor booleano indicando si la transferencia ha sido exitosa o no. Además emite un evento `Transfer`.  

```solidity
function allowance(address owner, address spender) external view returns(uint256);
```
* Función `allowance`: Devuelve el número de tokens restantes que el `spender` va a poder gastar en nombre del `owner` a través de la función `transferFrom`. Este valor cambia en función de la llamada a la función `approve` y `transferFrom`.

```solidity
function approve(address spender, uint256 amount) external returns(bool);
```
* Función `approve`: Asigna una cantidad (`amount`) de tokens que el `spender` puede gastar en nombre de la persona que ha llamado a la función. Devuelve un valor booleano si la operación ha tenido éxito. Además emite un evento `Approval`.

```solidity
function transferFrom(address sender, address recipient, uint256 amount) external returns(bool);
```
* Función `transferFrom`: Mueve una cantidad (`amount`) de tokens desde el `sender` hacia el `recipient` usando el mecanismo de autorización. Los tokens se restan de los tokens autorizados a gastar de la persona que llama a la función y de la cuenta del `spender`. Devuelve un booleano si todo ha ido bien. Además emite un evento `Transfer`.

```solidity
event Transfer(address from, address to, uint256 amount);
```
* Evento `Transfer`: Se emite cuando se mueven tokens de una cuenta (`from`) a otra (`to`). 

```solidity
event Approval(address owner, address spender, uint256 value);
```
* Evento `Approval`: Se emite cuando el `owner` de una cuenta autoriza a una cuenta `spender` a gastar una cierta cantidad de tokens (`value`).

A partir de estas definiciones, el token ERC20 se puede implementar de diferentes maneras siempre y cuando contenga como mínimo las funciones de la interfaz IERC20. Empresas como *Openzeppelin* han hecho su propia implementación, la cual es usada por desarrolladores como base para sus proyectos: [openzeppelin-contracts/blob/master/contracts/token/ERC20/ERC20.sol](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/ERC20.sol)

Vamos a ver un ejemplo de autorización:

1. Lo primero que hacemos es desplegar el contrato ERC20.sol, con todas las funciones implementadas. Le asignamos un supply de 1000.
```s
>>> token = ERC20.deploy(1000, {'from': accounts[0]})
Transaction sent: 0xc1f05b27e18ab68ac3478b5b04aa26d2607995276ad79a983db72f5ea5473606
  Gas price: 0.0 gwei   Gas limit: 12000000   Nonce: 0
  ERC20.constructor confirmed   Block: 1   Gas used: 474247 (3.95%)
  ERC20 deployed at: 0x3194cBDC3dbcd3E11a07892e7bA5c3394048Cc87
```

Bien, en esta implementación de la funcion `transfer` lo que se esta haciendo es primero, comprobar que al persona que esta realizando la transaccion tiene tokes suficiences (primer *require*)

## **2.** Providers, signers, ABIs y Approval Flows
### 2.1 Providers y signers
En Ethereum, para escribir o leer datos en la blockchain es necesario comunicarse a través de un nodo de Ethereum. Cada nodo tendra un estado dentro de la blockchain, que nos permitirá leer datos, mientras que si queremos escribirlos, será necesario el envio de transacciones.

Un `provider`, es una conexión a un nodo de Ethereum que nos permite leer datos. Se usarán cuando llamemos a funciones que solo lean datos, obtengan el balance de alguna cuenta, obtengan información sobre alguna transacción, etc.

Un `signer`, es una conexión a un nodo de Ethereum que nos permite escribir datos en la blockchain. Se usarán cuando queramos llamar a funciones que escriban datos, transferir ETH entre cuentas, etc. Para realizar todas este tipo de acciones el `signer` necesita acceder a una `private key` que usará para realizar transacciones.

|              | Provider | Signer |
| ------------ | :------: | :----: |
| **Leer**     |    X     |   X    |
| **Escribir** |          |   X    |

Por ejemplo, Metamask, inyecta un `provider` en nuestro navegador, de manera, que otras aplicaciones puedan utilizar ese mismo `provider` para leer datos de la blockchain donde nuestra billetera este conectada. Como Metamask no puede ir por ahi compartiendo nuestra clave privada, lo que hace es permitir a las dApps obtener un `signer`. De esta manera cuando queremos enviar una transaccion con metamask a través de la blockchain, la ventana de Metamask salta preguntando al usuario si quiere confirmar esa transacción.

### 2.2 BigNumbers
Cuando programamos con solidity, estamos muy acostumbrados a utilizar `uint256`, que tienen un rango que va desde `0` hasta `(2^256) - 1`, lo cual es un rango enorme. De normal se construyen interfaces con Javascript. El problema esque el tipo de dato `number` de Javascript tiene un limite mucho mas bajo que `uint256`.

Imaginemos que Javascript llama a una función de un smart contract que devuelve un `uint256` muchísimo mas grande que lo que soporta un numero en Javascript, bien, simplemente, no lo soporta. Para ello, existe un tipo especial denominado `BigNumber`, y librerias como `ethers.js` y `web3.js`, ya tienen integrado soporte para `BigNumber`.

`BigNumber` es una libreria para Javascript que implementa funciones matemáticas como `add`, `sub`, `mul`, `div`, etc.

### 2.3 ABI
Las siglas ABI provienen de **A**plication **B**inary **I**nterface.

Cuando se compila código de Solidity, lo hace bajo el bytecode, que a final de cuentas es binario. Este no contiene ni que funciones existen en el contrato, ni que parametros contiene, ni que valores devuelven. No obstante, si quieres llamar a una funcion de Solidity desde dentro de una aplicacion web, necesitas alguna manera de llamar al bytecode correcto, es decir, necesitas convertir el nombre de las funciones y parametros a bytecode y viceversa. 
La `ABI` nos permite precisamente conseguir esto. Cuando se compila un programa de Solidity, automaticamente se genera un fichero `ABI`. Este contiene metadata sobre funciones presentes en el contrato. 

Asi, cuando se quiera interactuar con un contrato, se necesitará tanto su dirección como su `ABI`. `ethers.js` por ejemplo, ya implementa esto, de manera que cuando nos comunicamos con un contrato, automaticamente codifica y decodifica las funciones a bytecode para poder comunicarnos con un nodo de Ethereum. 

A continuación se muestra un ejemplo de como se veria una funcion dentro de la `ABI`:

**Función *suma* dentro del fichero de Solidity:**
```solidity
function suma(uint num1, uint num2) public pure returns(uint) {
    return num1 + num2;
}
```

**Función *suma* dentro de la ABI:**
```json
{
    "inputs": [
    {
        "internalType": "uint256",
        "name": "num1",
        "type": "uint256"
    },
    {
        "internalType": "uint256",
        "name": "num2",
        "type": "uint256"
    }
    ],
    "name": "suma",
    "outputs": [
    {
        "internalType": "uint256",
        "name": "",
        "type": "uint256"
    }
    ],
    "stateMutability": "pure",
    "type": "function"
}
```

### 2.4 ERC20 Approval flow
Si queremos pagar o aceptar pagos de tokens ERC20, no es tan simple como llamar a una funcion `payable`. Esta función solo es buena para pagos de ETH. 

Para ello, el smart contract tendrá que, de alguna manera, substraer tokens de la persona que llame a dicha función. Aqui es cuando entra el flujo `Approve and transfer`.

Imaginemos el siguiente caso:
* Claudia quiere vender su NFT en su propia criptomoneda, ClaudiaCoin
* Ese NFT cuesta 10 ClaudiaCoin
* Pablo posee ClaudiaCoin y quiere comprarle el NFT
* Pablo necesita llamar a una función dentro del smart contract del NFT de Claudia, que cogerá sus 10 ClaudiaCoin y le dará el NFT
* Como los smart contracts no pueden aceptar un pago en ClaudiaCoin directamente, Claudia a implementado un flujo `Approval and transfer` dentro del contrato de su NFT.

Partimos de la base de que `ClaudiaCoin` es un token ERC20, que ya viene implementado con ciertas funciones relacionadas con el concepto de *Allowance* (autorización en español):

```solidity 
approve(address spender, uint256 amount)
```
Permite a un usuario `aprovar` a diferentes direcciones que gasten cierta `cantidad` de tokens en su nombre. Es decir, se le esta dando autorización al `spender` para gastar hasta un cierto `amount`.

```solidity 
transferFrom(address from, address to, uint256 amount)
```

Permite a un usuario enviar una `cantidad` de tokens desde `from` hasta `to`. Si el usuario que llama a la función, es el mismo que el de la dirección `from`, los tokens son eliminados de su balance. En caso de que el usuario se otro diferente a `from`, `from` deberá antes haber dado autorización al usuario para gastar una `cantidad` de tokens usando la función `approve`.

Volvemos al caso anterior:
* Pablo da la autorización al contrato del NFT de Claudia para gastar 10 de sus `ClaudiaCoins` usando la función `approve`
* Pablo llama a la función para comprar el NFT de Claudia en el contrato NFT de esta
* La función de comprar, internamente llama a `transferFrom` en `ClaudiaCoin` y transfiere 10 ClaudiaCoins desde la cuenta de Pablo a la cuenta de Claudia
* Como al contrato se le habia dado la autorización para gastar 10 ClaudiaCoins de la cuenta de Pablo, esta acción (de la función `transferFrom`) se permite
* Claudia por lo tanto recibe sus 10 ClaudiaCoins y Pablo su NFT

Es importante entender que el que llama a la función `transferFrom` es el contrato del NFT de Claudia, no Pablo.

Por lo tanto, para que pablo haya podido transferir 10 tokens a la cuenta de Claudia, ha tenido que realizar dos transacciones: **Transacción 1** - Dar el permiso al contrato. **Transacción 2:** - Llamar a la función de compra del contrato del NFT de Claudia.
