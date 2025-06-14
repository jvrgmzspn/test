# TeslaVehicleCommand

**TeslaVehicleCommand** is a .NET 8 library enabling secure, low‑level communication and command execution on Tesla vehicles via the official Tesla API.

## Features

- **Secure Session Establishment**: Implements ECDH handshake and HMAC‑based message authentication using BouncyCastle to derive shared keys and verify integrity. fileciteturn0file0
- **DomainSession**: Manages per‑domain session metadata (Handle, Counter, Epoch, ClockTime) and provides HMAC signing for Protobuf messages. fileciteturn0file0
- **SessionManager**: Orchestrates multiple DomainSession instances (e.g., Infotainment, VehicleSecurity), handles handshake requests, and attaches signatures to `RoutableMessage` objects. fileciteturn0file1
- **Protocol Buffers**: Leverages `Google.Protobuf` for efficient serialization of commands and responses. fileciteturn0file0turn0file1
- **Easy Integration**: Designed to plug into existing Tesla API workflows (e.g., TeslaAPI, \_Helpers). fileciteturn0file1

## Installation

Install via NuGet Package Manager:

```bash
dotnet add package TeslaVehicleCommand
```

Or search and install in Visual Studio:

```
Tools > NuGet Package Manager > Manage NuGet Packages for Solution...
```

## Usage Example: Open Rear Trunk

Below is a minimal console app demonstrating how to initialize a session and send a command to open the Tesla’s rear trunk. fileciteturn0file2

```csharp
using TeslaVehicleCommand;
using TeslaProtobuf.Vcsec;
using TeslaProtobuf.Universalmessage;
using _Helpers;
using Google.Protobuf;
using Org.BouncyCastle.Crypto;
using Org.BouncyCastle.OpenSsl;
using TeslaAPILib;

// Load your Tesla OAuth token and VIN
string token = "<YOUR_OAUTH_TOKEN>";
string vin = "<YOUR_VIN>";

// Initialize TeslaAPI client
var apiClient = new TeslaAPI(token);

// Load client's asymmetric key pair from PEM
AsymmetricCipherKeyPair clientKeyPair;
using (var reader = File.OpenText("path/to/client.key"))
{
    clientKeyPair = (AsymmetricCipherKeyPair)new PemReader(reader).ReadObject();
}

// Create SessionManager and start handshake
var sessionManager = new SessionManager(clientKeyPair, apiClient);
await sessionManager.StartSession(vin);

// Build a ClosureMoveRequest to open the rear trunk
var command = new UnsignedMessage();
command.ClosureMoveRequest = new ClosureMoveRequest
{
    RearTrunk = ClosureMoveType_E.ClosureMoveTypeOpen
};

// Wrap it in a RoutableMessage
var message = new RoutableMessage
{
    ToDestination = new Destination { Domain = Domain.VehicleSecurity },
    FromDestination = new Destination { RoutingAddress = ByteString.CopyFrom(RandomGenerator.Get()) },
    ProtobufMessageAsBytes = command.ToByteString(),
    Uuid = ByteString.CopyFrom(RandomGenerator.Get())
};

// Sign and send the message
sessionManager.SignRoutableMessage(message, Domain.VehicleSecurity);
string response64 = await apiClient.SendSignedCommandToVehicle(vin, message.ToByteString().ToBase64());

// Handle response
var response = RoutableMessage.Parser.ParseFrom(Convert.FromBase64String(response64));
// Check response.SignedMessageStatus.OperationStatus...
```

## API Reference

### DomainSession

- **Constructor**: `DomainSession(RoutableMessage response, string vin, AsymmetricCipherKeyPair clientKey, Domain domain)`
- **SignHMAC(ByteString protobufMessageAsBytes)**: Returns HMAC tag for the given Protobuf payload.

### SessionManager

- **Constructor**: `SessionManager(AsymmetricCipherKeyPair clientKey, TeslaAPI teslaApi)`
- **StartSession(string vin)**: Performs handshake to establish sessions for all supported domains.
- **SignRoutableMessage(RoutableMessage message, Domain domain)**: Attaches HMAC signature data to the message for the specified domain.

## Dependencies

- [BouncyCastle.Cryptography](https://www.nuget.org/packages/BouncyCastle.Crypto)
- [Google.Protobuf](https://www.nuget.org/packages/Google.Protobuf)
- [TeslaAPI](https://www.nuget.org/packages/TeslaAPI)
- [TeslaProtobuf](https://www.nuget.org/packages/TeslaProtobuf)
- [\_Helpers](https://www.nuget.org/packages/Helpers)

## Target Framework

- .NET 8.0

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

---

*For more information, visit the *[*TeslaVehicleCommand GitHub repository*](https://github.com/yourusername/TeslaVehicleCommand)*.*


