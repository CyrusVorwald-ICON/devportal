---
description: >-
  Sample -
  https://github.com/icon-project/iconex_ios/tree/master/docs/Connect/ConnectSample
---

# iOS

ICONex Connect is a simple protocol for supporting 3rd party applications who want send transactions, ICX or IRC tokens via ICONex wallet.

### Features

* Get address of ICON wallet which managed by ICONex.
* Request send transaction.
* Request send ICX or IRC Token.

### Basic Transmission Protocol

* Request

| Key | Type | Description |
| :--- | :--- | :--- |
| data | String | Base64 encoded string of JSON Object |
| caller | String | Custom URL Scheme |

```swift
var action = [String: Any]()
action["id"] = 1234
action["method"] = "bind"

guard let data = try? JSONSerialization.data(withJSONObject: action, options: []) else { return }
let encoded = data.base64EncodedString()

let items = [URLQueryItem(name: "data", value: encoded), URLQueryItem(name: "caller", value: "my-app-scheme://")]
var component = URLComponents(string: "iconex://")!
component.queryItems = items

UIApplication.shared.open(component.url!, options: [:], completionHandler: nil)
```

* Response

  `my-app-scheme://?data=Base64EncodedString`

### JSON Speicifications

* [Value Types](json-rpc-specification#section-value-types)

### API Convention

```swift
// Request
{
    "id": $INT1,
    "method": "$STRING1"
    "params": {
        "$KEY1": "$VALUE1",
        "$KEY2": "$VALUE2",
        ...
    }
}

// Response
{
    "id": $INT1,
    "code": $INT2,    // if code == 1 success, else fail
    "result": "$VALUE1"
}
```

| Key | Type | Description | Required |
| :--- | :--- | :--- | :--- |
| id | INT | Request Identifier | Required |
| method | String | Method for current request | Required |
| params | T\_DATA\_TYPE | Parameters for current request | Optional |

### Methods

| Method | Description | Required Parameters |
| :--- | :--- | :--- |
| bind | Request wallet address | - |
| sign | Request sign for transaction | version, from, value, stepLimit, timestamp, dataType\(optional\), data\(optional\) |
| sendICX | Request send ICX | from, to, value |
| sendToken | Request send IRC token | from, to, value, contractAddress |

#### bind

* Return selected wallet's address.

**Parameters**

NULL

**Returns**

Selected wallet's address.

**Example**

```swift
//Request
{
    "id": 1234,
    "method" : bind
}

// Response - success
{
    "id": 1234,
    "code": 1,
    "result": "hx1234..."
}

//Response - fail
{
    "id": 1234,
    "code": -1000,
    "result": "Operation canceled by user."
}
```

#### sign

* Returns the signature of the transaction hash.

**Parameters**

| Key | Type | Description | Required |
| :--- | :--- | :--- | :--- |
| version | T\_INT | Protocol version \("0x3" for v3\) | Required |
| from | T\_ADDR\_EOA | EOA address that created transaction | Required |
| to | T\_ADDR\_EOA | EOA address to receive coins, or SCORE address to execute the transaction. | Required |
| value | T\_INT | Amount of ICX coins in to transfer. When omitted, assumes 0. \(1 icx = 1 \* 10^18 loop\) | Required |
| stepLimit | T\_INT | Maximum Step allowance that can be used by the transaction. | Required |
| timestamp | T\_INT | Transaction creation time. timestamp is in microsecond. | Required |
| nid | T\_INT | Network ID \("0x1" for Mainnet, "0x2" for Testnet, etc\) | Required |
| nonce | T\_INT | An arbitrary number used to prevent transaction hash collision. | Required |
| dataType | T\_DATA\_TYPE | Type of data \(call, deploy or message\) | Optional |
| data | T\_DICT or String | The content of data varies depending on the dataType. | Optional |

**Returns**

Signature of transaction hash.

**Example**

```swift
// Request
{
    "id": 1234,
    "method": "sign",
    "params": {
        "version": "0x3",
        "from": "hxbe258ceb872e08851f1f59694dac2558708ece11",
        "to": "hx5bfdb090f43a808005ffc27c25b213145e80b7cd",
        "value": "0xde0b6b3a7640000",
        "stepLimit": "0x12345",
        "timestamp": "0x563a6cf330136",
        "nid": "0x2",
        "nonce": "0x1"
    }
}

// Response - success
{
    "id": 1234,
    "code": 1,
    "result": "SIGNED_VALUE"
}

// Response - fail
{
    "id": 1234,
    "code": -1000,
    "result": "Operation canceled by user."
}
```

#### sendICX

* Send ICX via ICONex and returns transaction hash.

**Parameters**

| Key | Type | Description | Required |
| :--- | :--- | :--- | :--- |
| from | T\_ADDR\_EOA | EOA address that will send ICX coins. | Required |
| to | T\_ADDR\_EOA | EOA address to receive coins. | Required |
| value | T\_INT | Amount of ICX coins in to transfer | Required |
| dataType | T\_DATA\_TYPE | Type of data \(call, deploy or message\) | Optional |
| data | T\_DICT or String | the content of data varies depending on the dataType. | Optional |

**Returns**

Transaction hash

**Example**

```swift
// Request
{
    "id": 1234,
    "method": "sendICX",
    "params": {
            "to": "hxab2d8215eab14bc6bdd8bfb2c8151257032ecd8b",
            "from": "hxbe258ceb872e08851f1f59694dac2558708ece11",
            "value": "0x1"
        }
}

// Response - success
{
    "id": 1234,
    "code": 1,
    "result": "0xb903239f8543d04b5dc1ba6579132b143087c68db1b2168786408fcbce568238"
}

// Response - fail
{
    "id": 1234,
    "code": -1000,
    "result": "Opertaion canceled by user."
}
```

#### sendToken

* Send IRC token via ICONex and returns transaction hash.

**Parameters**

| Key | Type | Description | Required |
| :--- | :--- | :--- | :--- |
| from | T\_ADDR\_EOA | EOA address that will send IRC token. | Required |
| to | T\_ADDR\_EOA | EOA address to receive tokens. | Required |
| value | T\_INT | Amount of IRC token | Required |
| contractAddress | T\_ADDR\_EOA | SCORE contract address | Required |

**Returns**

Transaction hash

**Example**

```swift
// Request
{
    "id": 1234,
    "method": "sendToken",
    "params": {
            "from": "hxbe258ceb872e08851f1f59694dac2558708ece11",
            "to": "hxab2d8215eab14bc6bdd8bfb2c8151257032ecd8b",
            "value": "0x1",
            "contractAddress": "cxb0776ee37f5b45bfaea8cff1d8232fbb6122ec32"
        }
}

// Response - success
{
    "id": 1234,
    "code": 1,
    "result": "0xb903239f8543d04b5dc1ba6579132b143087c68db1b2168786408fcbce568238"
}

// Response - fail
{
    "id": 1234,
    "code": -1000,
    "result": "Opertaion canceld by user"
}
```

### Error Code

| Code | Message | Description |
| :--- | :--- | :--- |
| -1000 | Operation canceld by user. | User cancel |
| -1001 | Parse error \(Invalid JSON type\) | Data is not a JSON type |
| -1002 | Invalid request. | JSON must contains required parameters. |
| -1003 | Invalid method | Invalid method |
| -1004 | Not found caller. | Could not found caller in request. |
| -2001 | ICONex has no ICX wallet. | ICONex has no ICX wallet for support. |
| -2002 | Not found parameter. \('$key'\) | $key is not found in "params" |
| -3001 | Could not find matched wallet. \('$address'\) | There is no wallet matched with given address. |
| -3002 | Sending and receiving address are same. | Sending and receiving address are same. |
| -3003 | Insufficient balance. | Requested value is bigger than balance. |
| -3004 | Insufficient balance for fee. | Insufficient balance for fee. |
| -3005 | Invalid parameter. \('$key'\) | Something is wrong with value. |
| -4001 | Failed to sign. | Error occurs while signing |
| -9999 | Somethings wrong with network \($message\) | All errors while networking. $message may contains the reason of error. |

