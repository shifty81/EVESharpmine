<h3 align="center"><img src="https://raw.githubusercontent.com/wiki/Almamu/EVESharp/assets/EVESharp.png" height="200px"></h3>

<p align="center">
	<a href="https://github.com/Almamu/EVESharp/blob/master/LICENSE"><img src="https://img.shields.io/github/license/Almamu/EVESharp" /></a>
	<a href="https://github.com/Almamu/EVESharp/pulse"><img src="https://img.shields.io/tokei/lines/github/Almamu/EVESharp" /></a>
	<a href="https://www.codefactor.io/repository/github/almamu/evesharp"><img src="https://img.shields.io/codefactor/grade/github/Almamu/EVESharp" /></a>
	<a href="https://github.com/Almamu/EVESharp/graphs/commit-activity"><img src="https://img.shields.io/github/commit-activity/w/Almamu/EVESharp" /></a>
	<a href="https://github.com/Almamu/EVESharp/graphs/contributors"><img src="https://img.shields.io/github/contributors/Almamu/EVESharp" /></a>
	<a href="https://discord.gg/pSyEVg2M"><img src="https://img.shields.io/discord/165291219205881856" /></a>
	<a href="https://github.com/Almamu/EVESharp/issues"><img src="https://img.shields.io/github/issues-raw/Almamu/EVESharp" /></a>
	<a href="https://github.com/Almamu/EVESharp/issues?q=is%3Aissue+is%3Aopen+label%3A%22helpwanted%22"><img src="https://img.shields.io/github/issues/Almamu/EVESharp/helpwanted?color=green" alt="help wanted"></a>
</p>

# Introduction
EVESharp (or EVE#) is an EVE Online Server Emulator written in C# for .NET Framework 5.0. This project started off as a rewrite of [EVEmu](https://github.com/evemuproject/evemu_apocrypha/).

The aim is to provide a cluster-based server compatible with EVE Online Aprocrypha 6.14.101786.

**EVESharp (or EVE#) is an educational project**. Our main objective is to learn more about C#, Network-based Applications and Protocols for large scale projects. This software is in no way intended for running public game servers whatsoever, and we do not support that in any way.

**The EVESharp team isn't responsible for the usage given to this software, it is provided AS IS without any kind of warranty, for more information please read [the license](LICENSE)**

For more information on how to use this project, setup, etc please look at our wiki [here](https://github.com/Almamu/EVESharp/wiki)

## Quick Start

### Prerequisites
- .NET 5.0 SDK or higher
- MySQL 8.0 or MariaDB 10.5+
- EVE Online Apocrypha client (build 101786)

### Database Setup
1. Download the database dump from https://files.evesharp.dev/apoc/apo15-mysql5-v1.sql.bz2
2. Extract and prepare the SQL files using scripts in `Database/Utils/`
3. Import in this order: official dump → Oficial.sql → Static.sql → Dynamic.sql → Server.sql
4. See `Database/README` for detailed instructions

### Running the Server
```bash
# Build the solution
dotnet build Server/EVESharp.sln

# Run in single-server mode (recommended for testing)
cd Server/EVESharp.Node
dotnet run -- --mode single
```

### Client Connection
The client must be configured to connect to your server's IP and port (default: 26000). The server uses the Macho protocol for authentication and game communication.

## Project Structure
- **Server/** - C# server implementation with all game services
- **Database/** - SQL schemas and setup utilities  
- **Tools/** - Development utilities and client-side scripts

## Documentation
- Database setup: `Database/README`
- Configuration: Review `Server/EVESharp.Node/Configuration/` classes
- Services: Explore `Server/EVESharp.Node/Services/` for game logic
- **Architecture options:** `ARCHITECTURE_OPTIONS.md` - Analysis of client approaches and standalone integration
- **Web client guide:** `WEB_CLIENT_GUIDE.md` - Step-by-step guide for building a web-based client
