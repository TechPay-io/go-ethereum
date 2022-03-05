**Go Ethereum over sirius**
Note

Repo is the github.com/ethereum/go-ethereum fork. It should be in the same local path as original: $GOPATH/src/github.com/ethereum/.
Aims

Ethereum over sirius network and consensus. Full ethereum stack and sirius performance.
Demo

    start sirius test net first: cd $GOPATH/src/github.com/TechPay-io/go-photon/demo && make start;
    change dir: cd sirius/demo;
    make geth docker image and run 2 containers: make && make start;
    try to send tx: ./20.txn.sh;
    try to get balanses: ./10.balances.sh;
    stop docker containers: make stop;
    stop sirius test net;

Changes

    Rename p2p.Server to p2p.p2pServer;
    Create p2p/interface.go (p2p.ServerInterface, p2p.Server struct, p2p.p2pServer's additionals methods, p2p.siriusAdapter interface);
    Create eth/sirius.go (p2p.SiriusAdapter implementation);
    Add p2p.Config.SiriusAdapter SiriusAdapter;
    Create SiriusAddrFlag in cmd/utils/flags.go:

        SiriusAddrFlag = cli.StringFlag{
    	Name:  "sirius",
    	Usage: "sirius-node address",
    }
    . . .
    func setListenAddress(ctx *cli.Context, cfg *p2p.Config) {
    	. . .
    	if ctx.GlobalIsSet(SiriusAddrFlag.Name) {
    		cfg.SiriusAdapter = eth.NewSiriusAdapter(ctx.GlobalString(SiriusAddrFlag.Name), cfg.Logger)
    	}
    }

    Append utils.SiriusAddrFlag to:
        nodeFlags in cmd/geth/main.go;
        AppHelpFlagGroups in cmd/geth/usage.go;
        app.Flags in cmd/swarm/main.go;
    Make node.Node create .server according to .serverConfig.SiriusAdapter and use p2p.Server.AddProtocols() at .Start():

    var running *p2p.Server
    if n.serverConfig.SiriusAdapter == nil {
    	running = p2p.NewServer(n.serverConfig)
    	n.log.Info("Starting peer-to-peer node", "instance", n.serverConfig.Name)
    } else {
    	running = sirius.NewServer(n.serverConfig)
    	n.log.Info("Using sirius node", "address", n.serverConfig.SiriusAdapter.Address())
    }
    . . .
    for _, service := range services {
    	running.AddProtocols(service.Protocols()...)
    }

    Create sirius/ package;

TODO:

    make ethereum blocks from sirius commits at eth.siriusAdapter without mining;
    switch sirius/demo/docker/Dockerfile.geth* from local to origin "github.com/TechPay-io/go-photon" when stable;
