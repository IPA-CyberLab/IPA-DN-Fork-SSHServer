
# SSHServer

SSH Server library based heavily on [FxSsh](https://github.com/Aimeast/FxSsh)

## FxSsh

FxSsh is a lightweight [SSH](http://en.wikipedia.org/wiki/Secure_Shell) server side application as SSH reinforcement of [GitCandy](https://github.com/Aimeast/GitCandy). 

### Features in this fork

- Password Authentication is implemented and enabled


## Sample

### Server (Powered by this library)

```csharp
using FxSsh;
using FxSsh.Services;
using System;
using System.Text;
using System.Threading.Tasks;

namespace SshServerLoader
{
    class Program
    {
        static void Main(string[] args)
        {
            var server = new SshServer(new SshServerSettings { Port = 4022 });
            server.AddHostKey("ssh-rsa", "<RSAKeyValue><Modulus>xXKzcIH/rzcfv2D7VcvLdxR5S5iw2TTsP65Aa82S4+9ZIqLPTNtuzr76Mz6Cx0yDOhawHlIujtalqaaQzaUvkudCtMcVMnj37OcCYz7XDAYejalCxf/vtJo7U4mnYdCM+nAOQKNIDKLGbLtGuEAGwdi560DOJY2plhnBf1oOI+k=</Modulus><Exponent>AQAB</Exponent><P>37kMr9YiU4cSqHqTJSjBJ/szG2O4n5xSIlPy4MZ4aAN5NALHxfsN0dq1y8NL6GTLMI5qoykvp4Bjrm2ZgU1cDQ==</P><Q>4e84rF+UsFBfQEKJc2pbACWWJjttNW0hccdQZzA3IUxRmd/Z4yEMr1L70TP0XV7dw1RDs1JyU7xnXBIbGy5ETQ==</Q><DP>vP0TbI6VnL3j0xMIrkFJOj8Ho0GQOrTQ5VLJP3wpRqR4hKk8nVBBEl+RZznpK73Jr5D/ICmwqezZSAYpwILbGQ==</DP><DQ>bHdzZtWwRYEgaXJIGL+7lnN1BT/MazTMNJpykEeGgBbqqgvcx/zq4RTezg26SEUuBANlSSbQukCeAoayurbYlQ==</DQ><InverseQ>Irq9vR7CXIVR+r09caYIxIY8BOig+HShN1bXvATERJcjTW2jUgJrUttDGNEx70/hBd7m1NWCZz5YO3RH9Bdf5w==</InverseQ><D>AqxsufxFcW9TDCAmQK4mwVdsoQjRp2jfcULmkM8fl9u40dtxTr6Csv5dz7qfKLWxHTGlDUDabCK2t/DCcZZoA3rsqwLADe4ZerDdg6xiq4MBzNprM8Y0IfNESEdFB9T0T73ONQCsMalUzEvUknC4Ed4Fya34LUHntgQtEhXpDJE=</D></RSAKeyValue>");
            server.AddHostKey("ssh-dss", "<DSAKeyValue><P>1fge/S/6Y42F73v/RhtkZQUEgNktLUzjf4zcJPse2JfXGqtg3lMaEiEDbb/2Nm4Q/M7iksZxlIxge0BkP1ul69+Tovl+Gg+hI+OPGaKlD/1dnX4D4i1FiOpnjtyoCTeZysJ5X8nO1lSInDRnRv46gHk4cHSFVKILKY7CVo1sf0s=</P><Q>tb2irBOnujteP+kf5HmH0yXepE8=</Q><G>HJfQK2Sdd5vN47HWVdmRRYhwAhiVoP98l7FZ3FzVxZvE/CurAL2xVrDi9modbLg0UA1JHeOi6h9FTI9ijepnqRscBKFlyIny0wh15sc8EMcZzG/vJVH2M48ps1knt793sxEwjU/SPYxE7UEX5HGa/KyLuP6Ev/0z4WOOIJ62B5w=</G><Y>ebWWjjGiFpfKfaXlrY3SiMorrkvge/UEeHd5E27RS4B/BBlm9OhdNBFq3zzILAa3X8KvDOGlVZgTYqDchp3/SK/Y8HsbqNyTyqDMNvcnHILDqSrWmGMQEA/0kGiPn0nwEcA87F1osCro7Y70SuLNwUOl3xh+gQKdi5V6WU75gaw=</Y><Seed>w0QY1PFWINvz3x2OZlVF6Vi5cCw=</Seed><PgenCounter>UQ==</PgenCounter><X>SFKXJnLYP8UM5tlOCpVQ0S2yE78=</X></DSAKeyValue>");
            server.ConnectionAccepted += OnConnectionAccepted;

            server.Start();

            Task.Delay(-1).Wait();
        }

        static void OnConnectionAccepted(object sender, Session e)
        {
            Console.WriteLine("Accepted a client.");

            e.ServiceRegistered += OnServiceRegistered;
        }

        static void OnServiceRegistered(object sender, SshService e)
        {
            var session = (Session)sender;
            Console.WriteLine("Session {0} requesting {1}.",
                BitConverter.ToString(session.SessionId).Replace("-", ""), e.GetType().Name);

            if (e is UserAuthService)
            {
                var service = (UserAuthService)e;
                service.UserAuth += OnUserAuth;
            }
            else if (e is ConnectionService)
            {
                var service = (ConnectionService)e;
                service.CommandOpened += OnServiceCommandOpened;
            }
        }

        static void OnUserAuth(object sender, UserAuthArgs e)
        {
            switch (e.AuthenticationType)
            {
                case UserAuthArgs.AuthType.PublicKey:
                    {
                        Console.WriteLine($"Client {e.KeyAlgorithm} fingerprint: {e.Fingerprint}.");
                        break;
                    }
                case UserAuthArgs.AuthType.Password:
                    {
                        Console.WriteLine($"Client {e.Username} password: {e.Password}");
                        break;
                    }
            }

            e.Result = true;
        }

        static void OnServiceCommandOpened(object sender, SessionRequestedArgs e)
        {
            Console.WriteLine("Channel {0} runs command: \"{1}\".", e.Channel.ServerChannelId, e.CommandText);
            e.Channel.SendData(Encoding.UTF8.GetBytes($"You ran {e.CommandText}\n"));
            e.Channel.SendClose();
        }
    }
}
```

### Client (powered by SSH.NET client)

```csharp
using Renci.SshNet;
using System;
using System.Net;

namespace TestClient
{
    internal class Program
    {
        private static void Main(string[] args)
        {
            var testUsername = "test_person";
            var testPassword = "1234";
            var connInfo = new ConnectionInfo(IPAddress.Loopback.ToString(), 4022, testUsername,
                new AuthenticationMethod[]
                {
                    // Password auth
                    new PasswordAuthenticationMethod(testUsername, testPassword)
                }
            );
            using (var sshClient = new SshClient(connInfo))
            {
                sshClient.Connect();
                var commands = new[] { "info", "hello", "rm -rf /" };
                foreach (var cmd in commands)
                {
                    using (var sshCmd = sshClient.CreateCommand(cmd))
                    {
                        sshCmd.Execute();
                        var result = sshCmd.Result;
                        Console.Write($"exec: {cmd}, result: {result}");
                    }
                }
            }
            Console.ReadLine();
        }
    }
}
```

### License

The MIT license
