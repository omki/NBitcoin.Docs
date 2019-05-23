# Connect to your own node via the P2P protocol

NBitcoin allows you to connect to the P2P protocol. The documentation of the P2P protocol can be found spread accross many [Bitcoin Improvement Proposals (BIPs)](https://github.com/bitcoin/bips/).

You can easily test the P2P protocol with the test framework
```csharp
using System;
using System.Threading;
using System.Threading.Tasks;
using NBitcoin;
using NBitcoin.Altcoins;
using NBitcoin.Protocol;
using NBitcoin.Protocol.Behaviors;
using NBitcoin.Tests;

namespace NBitcoinTraining
{
    class Program
    {
        // For this to build, you need to set C# 7.1 in your csproj file
        // <PropertyGroup>
        //   ...
        //   ...
        //   <LangVersion>7.1</LangVersion>
        // </PropertyGroup>
        static async Task Main(string[] args)
        {
            // During the first run, this will take time to run, as it download bitcoin core binaries (more than 40MB)
            using (var env = NodeBuilder.Create(NodeDownloadData.Bitcoin.v0_18_0, Network.RegTest))
            {
                // Removing node logs from output
                env.ConfigParameters.Add("printtoconsole", "0");
                var testNode = env.CreateNode();
                env.StartAll();

                var p2p = testNode.CreateNodeClient();
                p2p.VersionHandshake();
                System.Console.WriteLine("Connected via P2P!");
                testNode.Generate(101);

                var headerChain = p2p.GetSlimChain();
                System.Console.WriteLine($"Header chain retrieved with {headerChain.Height} blocks");
            }
        }
    }
}
```

Output:
```
Connected via P2P!
Header chain retrieved with 101 blocks
```

Outside of the test framework you would this instead to connect to your node:

```csharp
var p2p = await Node.ConnectAsync(Network.Main, "mynode.com");
```

You will see how to use the P2P protocol to retrieve blocks and transaction in the [Advanced Wallet Design part](AdvancedWalletDesign.md).

# Using Tor

For using the P2P protocol via Tor, you need to properly setup your SOCKS5 proxy configuration before connecting your node.

In this example, we assume you are running the Tor Browser locally. 

Running the Tor Browser will create a SOCKS proxy on port `9150`, which NBitcoin can use to route `.onion` traffic. This proxy sends the traffic through Tor.

By default, clearnet destination will not use Tor. If you want to force all traffic to use tor, set `SocksSettingsBehavior.OnlyForOnionHosts` to `false`.

```csharp
using System;
using System.Threading;
using System.Threading.Tasks;
using NBitcoin;
using NBitcoin.Altcoins;
using NBitcoin.Protocol;
using NBitcoin.Protocol.Behaviors;
using NBitcoin.Tests;

namespace NBitcoinTraining
{
    class Program
    {
        // For this to build, you need to set C# 7.1 in your csproj file
        // <PropertyGroup>
        //   ...
        //   ...
        //   <LangVersion>7.1</LangVersion>
        // </PropertyGroup>
        static async Task Main(string[] args)
        {
            var connectionParameter = new NodeConnectionParameters();
            // Assuming you are running the Tor browser locally
            var socksConfig = new SocksSettingsBehavior(Utils.ParseEndpoint("localhost", 9150));
            connectionParameter.TemplateBehaviors.Add(socksConfig);

            using (var node = await Node.ConnectAsync(Network.Main, "7xnmrhmkvptbcvpl.onion", connectionParameter))
            {
                node.VersionHandshake();
                Console.WriteLine("Connected!");
            }
        }
    }
}
```

Assuming `7xnmrhmkvptbcvpl.onion` is still online when you try this code,

Output:
```
Connected!
```