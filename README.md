# Solidity-notes

## Indice
### 1. **[Implementación ERC20](#implementación-erc20)**
* **[Interfaz IERC20](#interfaz-ierc20)**

### 2. **[Implementación ERC721](#implementación-erc721)**
* **[Interfaz ERC165](#interfaz-erc165)**
* **[Interfaz ERC721TokenReceiver](#interfaz-erc721tokenreceiver)**
* **[Interfaz IERC721](#interfaz-ierc721)**
* **[Implementación ERC721](#implementación-erc721)**

### 3. **[Providers, signers, ABIs y Approval Flows](#providers,-signers,-abis-y-approval-flows)**
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

1. Lo primero que hacemos es desplegar el contrato ERC20.sol, con todas las funciones implementadas. Le asignamos un supply de 1000 y cedemos todos los tokens a la cuenta `accounts[0]`.
```s
>>> token = ERC20.deploy(1000, {'from': accounts[0]})
Transaction sent: 0xc1f05b27e18ab68ac3478b5b04aa26d2607995276ad79a983db72f5ea5473606
  Gas price: 0.0 gwei   Gas limit: 12000000   Nonce: 0
  ERC20.constructor confirmed   Block: 1   Gas used: 474247 (3.95%)
  ERC20 deployed at: 0x3194cBDC3dbcd3E11a07892e7bA5c3394048Cc87
```

```s
>>> token.totalSupply()
1000
>>> token.balanceOf(accounts[0])
1000
```

1. Ahora `account[0]` hara una transferencia de 100 tokens a la cuenta `account[1]` y `account[2]`.

```s
>>> token.transfer(accounts[1], 100, {'from': accounts[0]})
Transaction sent: 0x0dc4d73200744953a5f72335933ec3e63ab46e1713f9729f9e55ac9e39979ebf
  Gas price: 0.0 gwei   Gas limit: 12000000   Nonce: 1
  ERC20.transfer confirmed   Block: 2   Gas used: 51932 (0.43%)

<Transaction '0x0dc4d73200744953a5f72335933ec3e63ab46e1713f9729f9e55ac9e39979ebf'>
>>> token.transfer(accounts[2], 100, {'from': accounts[0]})
Transaction sent: 0x68557d3de8dd80fbb265469f99cb243a6192c9fd02ba1e4e7055b24e434ffd59
  Gas price: 0.0 gwei   Gas limit: 12000000   Nonce: 2
  ERC20.transfer confirmed   Block: 3   Gas used: 51920 (0.43%)

<Transaction '0x68557d3de8dd80fbb265469f99cb243a6192c9fd02ba1e4e7055b24e434ffd59'>
>>> token.balanceOf(accounts[1])
100
>>> token.balanceOf(accounts[2])
100
```

3. En este punto tenemos la siguiente situación:
   * Balance de `accounts[0]`: 800 
   * Balance de `accounts[1]`: 100
   * Balance de `accounts[2]`: 100  

4. Ahora `accounts[1]` va a dar permiso a `accounts[2]` para gastar hasta 50 tokens.
```s
>>> token.approve(accounts[2], 50, {'from': accounts[1]})
Transaction sent: 0x698072c464a9c4c8ff9e5ea71e6fde00232c0c40a6b5ea0d9fd1552de6a781ef
  Gas price: 0.0 gwei   Gas limit: 12000000   Nonce: 0
  ERC20.approve confirmed   Block: 4   Gas used: 43983 (0.37%)

<Transaction '0x698072c464a9c4c8ff9e5ea71e6fde00232c0c40a6b5ea0d9fd1552de6a781ef'>
>>> token.allowance(accounts[1], accounts[2])
50
```
5. Lo que va a pasar ahora es que cuando `accounts[2]` ejecute la funcion transferFrom, poniendo a `accounts[1]` como la dirección que envia los tokens, estos se van a restar del balance de `accounts[1]` y no del suyo.
```s
>>> token.transferFrom(accounts[1], accounts[0], 25, {'from': accounts[2]})
Transaction sent: 0xef4d6dc50fb79e9a393cf634731623d57e8174fad91ce392859eceada66314ce
  Gas price: 0.0 gwei   Gas limit: 12000000   Nonce: 0
  ERC20.transferFrom confirmed   Block: 5   Gas used: 44670 (0.37%)

<Transaction '0xef4d6dc50fb79e9a393cf634731623d57e8174fad91ce392859eceada66314ce'>
```
6. Ahora por lo tanto los balances se quedan tal que así:
   * Balance de `accounts[1]`: 75, ya que `accounts[2]` ha gastado 25 tokens a su nombre.
   * Balance de `accounts[2]`: 100, seguira siendo 100 ya que realmente el no ha gastado nada de su balance.
   * Balande de `accounts[0]`: 825, es decir, los 800 que le quedaban mas los 25 que le ha transferido `accounts[2]`

7. Por último, tener en cuenta que ahora a `accounts[2]` solo le quedan 25 tokens para gastar en nombre de `accounts[1]` de los 50 que tenia.

```s
>>> token.allowance(accounts[1], accounts[2])
25
```

## **2.** Implementación ERC721

### 2.1 Interfaz ERC165
El estandar ERC165 es una manera de comprobar que el `fingerPrint` de un contrato coincide con el `fingerPrint` de una interface dada. La interface ERC165 solo contiene una funcion, `supportsInterface`:

```solidity
interface ERC165 {
  function supportsInterface(bytes4 interfaceID) external view returns(bool);
}
```

Por lo tanto, el contrato que implemente esta interface, (en este caso el ERC721) deberá implementar una función `supportsInterface` que cogerá un `interfaceId` y devolverá `true` si esa interface es soportada. Siendo `interfaceId` definido como *la operación XOR entre los selectores de todas las funciones en la interface*. 

Cuando hablamos de selectores nos referimos a los primeros 4 bytes del hash de la función (*nombre_funcion(tipo_argumento_1, ..., tipo_argumento_n)* Nada más), es por eso que el argumento `interfaceId` es de tipo `bytes4`. Hay dos maneras de obtener este selector:

1. 
```solidity
this.nombre_funcion.selector
```
2. 
```solidity
bytes4(keccak256("nombre_funcion(tipo_argumento)"))
``` 
Recordemos que al selector le da igual el nombre de los parametros, modificadores, mutabilidad, returns o el contenido de la función. 

Ahora imaginemos que una interface consiste en tres funciones diferentes: `function1()`, `function2()`, `function3()`. En este caso, el `interfaceId` seria el siguiente:

```solidity
interfaceId = this.function1.selector ^ this.function2.selector ^this.function3.selector;
``` 

Por lo tanto, ahora vamos a ver como implemetamos esta interfaz ERC165 para que nuestro contrato ERC721 herede de ella y podamos ver que interfaces soporta. Lo que se hace, es guardar las interfaces soportadas en un mapping, para ahorrarnos tener que calcular todo el rato los `interfacesId`.

```solidity
contract CheckERC165 is ERC165 {
  mapping (bytes4 => bool) internal supportedInterfaces;

  constructor() {
    supportedInterfaces[this.supportsInterface.selector] = true;
  }

  function supportsInterface(bytes4 interfaceId) external view returns(bool) {
    return supportedInterfaces[interfaceId];
  }
}
```

La primera interface que deberá poder soportar es la interface ERC165, es por eso que en propio constructor de `CheckerERC165` se añade el selector de esta interface. 

### 2.2 Interfaz ERC721TokenReceiver
Realmente el estandar ERC721 contiene cuatro interfaces diferentes:
* Una para el contrato general ERC721
* Otra para contratos que pueden recibir tokens ERC721
* Otras dos para extensiones opcionales que añaden funcionalidades extra.

Vamos a ver ahora esa segunda interface. No es obligatorio que nuestro token ERC721 herede de esta interface y como el nombre indica sirve para aquellos contratos que tienen que recibir tokens ERC721. La funcion que implementa esta interface `onERC721Received` se utiliza cuando se transfiere un token via `ERC721.safeTransferFrom` y se encarga de devolver su propio selector, es decir, `bytes4(keccak256("onERC721Received(address, address, uint256, bytes)"))`. Cualquier otro valor que se devuelva que no sea ese, hara que no se llegue a realizar la transferencia y esta sera revertida.

### 2.3 Interface IERC721

Vamos a ver cuales son las funciones que implementaremos luego en el punto **2.4**:

```solidity
function balanceOf(address owner) external view returns(uint256 balance);
```
* Función `balanceOf`: Permite ver la cantidad de tokens que posee una direción `owner`.

```solidity
function ownerOf(uint256 tokenId) external view returns(address owner);
```
* Función `ownerOf`: Permite saber quien es el dueño de cierto token mediante su identificativo `tokenId`.

```solidity
function safeTransferFrom(address from, address to, uint256 tokenId) external;
```
* Función `safeTransferFrom`: Permite realizar la transferencia de un token desde una cuenta `from` hacia una cuenta `to`. Los requerimientos para esta función son los siguientes:
  * `from`: No puede ser cero
  * `to`: No puede ser cero
  * `tokenId`: El token tiene que existir y que su dueño sea `from`
  * Si el que llama a la función no es `from`, se tiene que comprobar, que este le ha autorizado a transferir ese token mediante `approve` o `setApprovalForAll`
  * Si `to` hace referencia a un smart contract, este tiene que implementar `IERC721Receiver.onERC721Received`, a la cual se llama durante la ejecucion de safe transfer
  
  Emite un evento Transfer. 

```solidity
function safeTransferFrom(address from, address to, uint256 tokenId, bytes data) external;
```

* Función `safeTransferFrom`: Exactamente igual que `safeTransferFrom` pero tambien permite añadir datos.

```solidity
function transferFrom(address from, address to, uint256 tokenId) external;
```

* Función `transferFrom`: Permite realizar la transferencia de un token desde una cuenta `from` hacia una cuenta `to`. La persona que llama a la función tiene que asegurarse que la cuenta que recibe el token puede hacerlo. Es mucho menos segura que `safeTransferFrom` y esta ultima es la que se deberia utilizar en cualquier caso. 

```solidity
function approve(address to, uint256 tokenId) external;
```

* Función `approve`: Da permisos a la dirección `to` para transferir el token con el `tokenId` a otra cuenta. Este permiso se limpia cuando el token es transferido a otra cuenta (mediante el uso de `address(0)`). Un token solo puede tener este tipo de permisos para una cuenta. Requerimientos de esta función:
  * La cuenta que llama a esta función tiene que ser el dueño del token `tokenId` o ser un operador aprobado por el dueño del token.
  * El token `tokenId` tiene que existir   

  Emite un evento Approval.


```solidity
function setApprovalForAll(address operator, bool _approved) external;
```

* Función `setApprovalForAll`: Aprueba o elimina a un operador como operador para la cuenta a la que llama a la función. Si se aprueba, el operador seria capaz de llamar a la función `transferFrom` o `safeTransferFrom` para cualquier token del dueño. Requerimientos para esta función: 
  * El `operator` no puede ser el owner
  
  Emite un evento ApprovalForAll.

```solidity
function getApproved(uint256 tokenId) external view returns(address operator);
```

* Función `getApproved`: Devuelve el operador al que el dueño del token `tokenId` ha dado permisos.

```solidity
function isApprovedForAll(address owner, address operator) external view returns(bool);
```

* Función `isApprovedForAll`: Devuelve si el `operator` tiene permisos para todos los tokens del `owner`.



### 2.4 Implementación token ERC721

A la hora de implementar el estandar ERC721 vamos a seguir las siguientes premisas:
1. Se establecerá un supply de tokens durante la creación del token, los cuales, pertencerán todos al creador.
2. Una función, que solo se podra llamar por el creador, permitirá generar mas tokens cuando se llame. Estos nuevos tambien pertenecerán al creador en un inicio. 

Vamos a empezar con la implementación, para ello, comenzaremos con el inicio del fichero tal que asi:

```solidity
pragma solidity ^0.8.0;

import "./CheckERC165.sol";
import "./interfaces/IERC721.sol";
import "./ERC721TokenReceiver.sol";
import "./libraries/SafeMath.sol";
```

Como hemos dicho antes, se necesita la implementación de ERC165, la interface IERC721, la interfaz ERC721TokenReceiver y SafeMath para cuando tengamos que hacer calculos.

```solidity
contract TokenERC721 is ERC721, CheckERC165 {
  using SafeMath for uint256;

  address internal creator;

  // Cantidad de tokens que tiene cada cuenta
  mapping (address => uint256) internal balances;

  //Dado un tokenId, que dirección es su owner
  mapping (uint256 => address) internal owners;

  //Dado un tokenId, a que cuenta a dado permisos el dueño
  mapping (uint256 => address) internal allowances;

  //Dada una dirección, que direcciones tienen permitido usar todos sus tokens
  mapping (address => mapping(address => bool)) internal authorised;
}
```

Esos serian las principales variables que necesitariamos para gestionar el token, podrian haber mas si queremos implementar mas funcionalidades, pero esto es para las básicas. Ahora vamos con el constructor:

```solidity
contract TokenERC721 is ERC721, CheckERC165 {
  constructor(uint256 _initialSupply) {
    creator = msg.sender;

    balances[msg.sender] = _initialSupply;

    //ERC165 interface check
    supportedInterfaces[
      this.balanceOf.selector ^
      this.ownerOf.selector ^
      bytes4(keccak256("safeTransferFrom(address,address,uint256)")) ^
      bytes4(keccak256("safeTransferFrom(address,address,uint256,bytes)")) ^
      this.transferFrom(address,address) ^
      this.approve.selector ^
      this.setApprovalForAll.selector ^
      this.getApproved.selector ^
      this.isApprovedForAll.selector
    ] = true;
  }
}
```

Es decir, estamos cogiendo todas las funciones de la interfac IERC721 anteriormente explicada, y calculando su `interfaceId` mediante la operacion XOR entre todas ellas. Se ha utilizado la estructura `bytes4(...` en las funciones `safeTransferFrom` debido a que si nos, se produce lo que se conoce como *overload* en esta función, por lo tanto, si lo intentamos hacer mediante `.selector` lanzaria un `TypeError`. El *overload* se produce cuando existen varias funciones con exactamente el mismo nombre pero diferente cantidad de parametros.

```solidity
function _exist(uint256 _tokenId) internal view returns(bool) {
  return owners[_tokenId] != address(0);
}
```

Esta función va a ser necesaria para las siguientes que vamos a implementar, simplemente comprueba que un token exista, esto quiere decir, que se le haya asignado un dueño.

Vamos a ver la implementación de las primeras funciones:

```solidity
function balanceOf(address _owner) public view returns(uint256 balance){
  return balances[_owner];
}

function ownerOf(uint256 _tokenId) public view returns(address owner){
  require(_exist(_tokenId), "ERROR: El token no existe");
  return owners[_tokenId];
}
```

## **3.** Providers, signers, ABIs y Approval Flows
### 3.1 Providers y signers
En Ethereum, para escribir o leer datos en la blockchain es necesario comunicarse a través de un nodo de Ethereum. Cada nodo tendra un estado dentro de la blockchain, que nos permitirá leer datos, mientras que si queremos escribirlos, será necesario el envio de transacciones.

Un `provider`, es una conexión a un nodo de Ethereum que nos permite leer datos. Se usarán cuando llamemos a funciones que solo lean datos, obtengan el balance de alguna cuenta, obtengan información sobre alguna transacción, etc.

Un `signer`, es una conexión a un nodo de Ethereum que nos permite escribir datos en la blockchain. Se usarán cuando queramos llamar a funciones que escriban datos, transferir ETH entre cuentas, etc. Para realizar todas este tipo de acciones el `signer` necesita acceder a una `private key` que usará para realizar transacciones.

|              | Provider | Signer |
| ------------ | :------: | :----: |
| **Leer**     |    X     |   X    |
| **Escribir** |          |   X    |

Por ejemplo, Metamask, inyecta un `provider` en nuestro navegador, de manera, que otras aplicaciones puedan utilizar ese mismo `provider` para leer datos de la blockchain donde nuestra billetera este conectada. Como Metamask no puede ir por ahi compartiendo nuestra clave privada, lo que hace es permitir a las dApps obtener un `signer`. De esta manera cuando queremos enviar una transaccion con metamask a través de la blockchain, la ventana de Metamask salta preguntando al usuario si quiere confirmar esa transacción.

### 3.2 BigNumbers
Cuando programamos con solidity, estamos muy acostumbrados a utilizar `uint256`, que tienen un rango que va desde `0` hasta `(2^256) - 1`, lo cual es un rango enorme. De normal se construyen interfaces con Javascript. El problema esque el tipo de dato `number` de Javascript tiene un limite mucho mas bajo que `uint256`.

Imaginemos que Javascript llama a una función de un smart contract que devuelve un `uint256` muchísimo mas grande que lo que soporta un numero en Javascript, bien, simplemente, no lo soporta. Para ello, existe un tipo especial denominado `BigNumber`, y librerias como `ethers.js` y `web3.js`, ya tienen integrado soporte para `BigNumber`.

`BigNumber` es una libreria para Javascript que implementa funciones matemáticas como `add`, `sub`, `mul`, `div`, etc.

### 3.3 ABI
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

### 3.4 ERC20 Approval flow
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
