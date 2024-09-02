# BlackJack Game

Welcome to my BlackJack Game, a full-stack application consisting of a server-side application and a client-side application. The project has been divided into two separate repositories, included as Git submodules:

1. **BlackJackServer**: The server-side application built with .NET 6. It manages multiple game sessions and can be run using any .NET-supporting platform, not just Visual Studio 2022.
2. **BlackJackClient**: The client-side application that offers a smooth, interactive gaming experience and allows joining live multiplayer games.

Each submodule has its own independent Git history and its prerequisites for setup.

## Getting Started

To clone this main repository along with its submodules, run:
```
git clone --recursive https://github.com/username/blackjack-game.git
```

If you've already cloned the repository without the `--recursive` option, you can still pull the submodules separately by running:

```
git submodule init
git submodule update
```


## BlackJackServer Setup

Before getting started with BlackJackServer, ensure you have met the following requirements:

* The latest version of [.NET 6](https://dotnet.microsoft.com/download/dotnet/6.0) installed

After cloning the main repository, navigate to the BlackJackServer directory:
```
cd blackjack-game/BlackJackServer
```


To start the server, use any .NET-supporting platform to run the application.

## BlackJackClient Setup

Before getting started with BlackJackClient, ensure you have met the following requirements:

* The latest version of [Node.js and npm](https://nodejs.org/en/download/) installed

After cloning the main repository, navigate to the BlackJackClient directory:

```
cd blackjack-game/BlackJackClient
```

Install all the required packages and dependencies with npm:
```
npm install
```

After successfully installing the packages, you can start the development server by running:
```
npm run dev
```

## Game Guidelines

Here are some quick guidelines for our game:

- This is a multiplayer game designed for multiple sessions. To enjoy the game, open different tabs in your browser, each representing a unique player.
- After all the players have joined the room, initiate the game by clicking the 'Start Game' button in any one of the tabs.
- Your player indicator will be highlighted in blue.
- The player whose turn it is will have their cards glowing.

Enjoy your game!

## Individual Repositories

If you want to clone the submodules separately, you can find them at the following links:

- BlackJackClient: [https://github.com/AdamOliver1/BlackJackClient](https://github.com/AdamOliver1/BlackJackClient)
- BlackJackServer: [https://github.com/AdamOliver1/BlackJackServer](https://github.com/AdamOliver1/BlackJackServer)






