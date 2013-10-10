*By [Eric Terpstra][1]*

With the release of the Ouya, Xbox One and PS4 this year, couch-based console
gaming appears to be as popular as ever. Despite the proliferation of multi-
player games available on mobile devices, and even the advent of multi-player 
experiences available on the web, sitting next to the person you’re playing a 
game with is still an irreplaceable experience. While experimenting with Node.js
and the Socket.IO library, I found a perfect opportunity to not only learn some 
interesting new technologies, but also experiment with using the web and common 
devices (computers and mobile phones) to replicate a console-like gaming 
experience.

This article will give a brief overview of the fundamental concepts of the
Socket.IO library in the context of building a multi-player, multi-screen word 
game. A web browser on a large screen device, such as a TV or computer will be 
the ‘console’ and mobile browsers will act as the ‘controllers’. Socket.IO and 
Node.js will provide the necessary wiring for all the browsers to share data and
provide a cohesive, real-time game experience.

## The Game – Anagrammatix

The game itself is a made up challenge involving anagrams – words that can
have their letters re-arranged to form new words. The gameplay is quite simple
– on a large ‘Host’ screen, a word is displayed. On each player’s controller (
phone) is a list of similar words. One of the words is an anagram of the word on
the Host screen. The player to tap the anagram first gets points for the round. 
However, tapping an incorrect word will lose points for the player. After 10 
rounds, the player with the most points wins!

To download the code and play locally, first ensure that Node.js is installed.
Visit the[Anagrammatix GitHub repo][2], clone the repository with 
`git clone https://github.com/ericterpstra/anagrammatix.git` and then run 
`npm install` to download and install all the dependencies (Socket.IO and
Express). Once that is complete, run`node index.js` to start the game server.
Visit http://localhost:8080 in your browser to start a game. In order to test 
out the host as well as multiple “player” clients you’ll need to open multiple 
windows.

If you want to play with mobile devices, you will need to know the I.P. address
of your computer on your local network. Your mobile devices will also need to be
on the same network as the computer running the application. To begin, simply 
replace ‘localhost’ in the URL with the I.P. address (e.g. http://192.168.0.5:
8080). If you are on Windows, you may need to disable Windows firewall, or 
ensure port 8080 is open.

## The Underlying Technology

Anagrammatix is implemented using only HTML, CSS and JavaScript. To keep things
as simple as possible, the use of libraries and frameworks was kept to a minimum.
The major technologies used in the game are listed below:

*   **Node.js** - Node provides the foundation for the back-end portion of the
    game, and allows the use of the Express framework, and the Socket.IO library. 
    Also, some of the game logic for Anagrammatix takes place on the server. This 
    logic is contained in a custom
     *Node Module* which will be discussed later.
*   **Express** - On its own, Node.js does not provide many of the necessary
    requirements for creating a web application. Adding Express makes it much 
    simpler to serve static files such as HTML, CSS and JavaScript. Express is also 
    used to adjust logging output, and create an easy-to-use environment for Socket.
    IO
   
*   **Socket.IO** - Socket.IO makes it dead simple to open a real-time
    communication channel between a web browser and a server (in this case, a server
    running Node.js and Express). By default, Socket.IO will use the
     *websockets* protocol if it is supported by the browser. Older browsers
    such as IE9 do not support websockets. In that case, Socket.IO will fall back to
    other technologies, such as using Flash sockets or an Ajax technique called
     *long-polling*.
*   Client side JavaScript – On the client side, a few libraries are used to
    make things a little easier:
   
    *   jQuery – Mainly used in this case for DOM manipulation and event
        handling.
       
    *   [TextFit ][3]- A small library that will expand text to fit the size of
        its container. This is what allows the ‘Anagrammatix’ title screen to fill the 
        entire width of the browser no matter the screen size.
       
    *   [FastClick.js][4] – A small library that helps remove the 300ms delay
        on mobile devices when ‘tapping’ the screen. This makes the game controller 
        slightly more responsive when playing.
       
*   CSS – Some standard CSS is used to add a little color and style to the
    game.
   

## The Architecture

![architecture][5]

The basic setup for the game follows the recommended configuration for using
Socket.IO with Express on the[Socket.IO website][6]. In this configuration,
Express is used to handle HTTP requests coming into the Node application. The 
Socket.IO module attaches itself to Express and listens on the same port for 
incoming websocket connections.

When a browser connects to the application, it is served index.html and all the
required JavaScript and CSS to begin. The JavaScript in the browser will 
initiate a connection to Socket.IO and create a websocket connection. Each 
websocket connection has a unique identifier (uid) so Node and Socket.IO can 
keep track of which communications to send to which browser.

## The Code – Server Side Setup

The Express and Socket.IO modules do not come packaged with Node.js. they are
external dependencies that must be downloaded and installed separately. Node 
Package Manager (npm) will do that for us with the`npm install` command as long
as all the dependencies are listed in the package.json file. You can find 
package.json in the root of the project folder, and it contains the following 
JSON snippet to define the project dependencies:

    "dependencies": {
        "express": "3.x",
        "socket.io":"0.9"
    }

The [index.js][7] file in the root of the project is the main entry point for
the entire application. The first few lines instantiate all the required modules.
Express is configured to serve static files, and Socket.IO is set up to listen 
on the same port as Express. The following lines show the bare minimum to get 
the server going.

    // Create a new Express application
    var app = express();
    
    // Create an http server with Node's HTTP module. 
    // Pass it the Express application, and listen on port 8080. 
    var server = require('http').createServer(app).listen(8080);
    
    // Instantiate Socket.IO hand have it listen on the Express/HTTP server
    var io = require('socket.io').listen(server);

All of ther server-side code specific to the game is in [agxgame.js][8]. This
file is pulled into the application as a Node module with
`var agx = require('./agxgame')`. When a client connects to the application via
Socket.IO, the`initGame` function of the agxgame module should run. The
following code will make that happen.

    io.sockets.on('connection', function (socket) {
        //console.log('client connected');
        agx.initGame(io, socket);
    });

The `initGame` function in the agxgame module will set up the *event listeners*

    gameSocket.on('hostCreateNewGame', hostCreateNewGame);

In the code above, the `gameSocket` is a reference to an object created by
Socket.IO to encapsulate functionality regarding the unique socket connection 
between the server and the connected browser. The`on` function creates a
listener for a specific event and binds it to a function. So when the browser*
emits* an event named `hostCreateNewGame` through the websocket, Socket.IO will
then execute the`hostCreateNewGame` function. Names of events and functions don
’t have to be the same, it is just a nice convenience.

## The Code – Client Side Setup

When a web browser connects to the game, it is served index.html from contents
of the “public” folder, which consists of an empty div with an id of*gameArea*
`<script type=text/template">` tags. These bits of HTML are substituted
into the*gameArea* depending on what needs to be displayed on screen.

The Socket.io client library is required for the browser to connect to the
Socket.IO server. Because we are serving Socket.IO using Express, we can use an
*Express route* to request the library files from the server using the
following script tag:

    <script src="/socket.io/socket.io.js"></script>

The majority of the game logic, and all the client-side code is stored within
[app.js][9]. The code is enclosed within a self executing function, and is
organized using an*object literal namespacing* pattern. This means that all the
variables and functions used in the application are properties of the`IO` and
`App` objects. The basic outline for the client-side code is as follows:

    // Enclosing function
    function() {
    
        IO {
            All code related to Socket.IO connections goes here.
        }
    
        App {
            Generic game logic code.
    
            Host {
                Game logic for the 'Host' (big) screen.
            }
    
            Player {
                Game logic specific to 'Player' screens.
            }
        }       
    }

When the application first loads, there are two functions that execute when the
document is ready
– `IO.init()` and `App.init()`. The first will set up the Socket.IO connection
, and the latter will display the title screen in the browser.

The following line of code in `IO.init()` will initiate a Socket.IO connection
between the browser and the server:

    IO.socket = io.connect();

Following that, a call is made to `IO.bind()`, which sets up all the event
listeners for Socket.IO on the client. These event listeners work just like 
those that we set up on the server, but work in the opposite direction. Take the
following example:

    IO.socket.on('playerJoinedRoom', IO.playerJoinedRoom );

In the above line of code, `IO` is a container object used for organizing code
, the`socket` object is created by Socket.IO and has properties and methods
related to the websocket connection, and the`on` function creates an event
listener. In this case, when the server emits an event labeled
`playerJoinedRoom`, then the `IO.playerJoinedRoom` function is executed on the
client.

## Client-Server Communication

The gameplay of Anagrammatix is quite simple. There are not many rules, no
graphics or animation, and there are never more than a few words on any screen 
at a time. What makes this application a cohesive game and (arguably) fun is the
communication between the three browser windows. It’s important to re-iterate 
that browsers do not communicate directly with each other, but rather emit data 
to the server, which is then processed and re-emitted back to the appropriate 
browsers. Each of these events carries data with it, so information from the 
client browser such as the player name or selected answer can travel to the 
server and make its way into other clients.

All of this data flowing from client to server and back again can be a bit
difficult to follow, especially with three clients connected at once. Luckily 
there is a great tool available in Google’s Chrome browser to help out. If you 
open the Chrome developer tools window and go to the*Network* tab, you will be
able to monitor all websocket traffic for that particular window by clicking the
*WebSockets* button along the bottom toolbar. Within the WebSockets panel, a
list of connections appear in the left-hand column. Clicking on the connection, 
and then clicking the*Frames* tab will display a list of communications that
have occurred through that particular websocket connection. See the screenshot 
below for an example

![chrome_network][10]

The first column of the *Frames* tab is the most interesting. It shows the data
that is passed through the websocket connection. In the screenshot above, each 
object in the data column represents and event that was emitted. Each object has
a “name” and an “args” property. The value for the*name* is the name of the
event being emitted, such as`playerJoinedRoom` or `hostCheckAnswer`. The *args*

The Frames tab in Chrome developer tools, along with liberal use of 
`console.log()` makes it a bit easier to debug the application and keep track
of events and data passing between clients and the server.

## The Gameplay

Now that we have a brief overview of how the game is set up and how the
underlying architecture works, let’s step through the actual gameplay and trace 
the flow of a single playthrough of a game of Anagrammatix.

### Open a Browser and Visit the Application URL

**Node.js and Express serve the files from the public folder.** The index.html
file is loaded, along with[app.js][9] and the required client-side JavaScript
libraries.

**`IO.init()` and `App.init()` are called in [public/app.js][11].** The client
side game code is initialized. Event handlers are set up for Socket.IO, and code
is initiated to load the title screen. The FastClick lib is loaded to speed up 
touch events. The following code in the`App.showInitScreen` function will load
the`intro-screen-template` from index.html into the *gameArea* div so it is
visible to the user.

    App.$gameArea.html(App.$templateIntroScreen);

Also, the `App.bindEvents` function will create a number of click handlers for
buttons that appear on-screen. The following lines of code create click handlers
for the CREATE and JOIN buttons that appear on the title screen.

    App.$doc.on('click', '#btnCreateGame', App.Host.onCreateClick);
    App.$doc.on('click', '#btnJoinGame', App.Player.onJoinClick);

### The Create Button is Clicked

**The `hostCreateNewGame` event is emitted by the client.** The function 
`App.Host.onCreateClick` does one thing – emit an event to the server to
indicate that a new game must be created:`IO.socket.emit('hostCreateNewGame')`

**The server creates a new *room* for the game, and emits the room ID back to
the client.** The concept of a *room* is built into Socket.IO. A room will
collect specific client connections and allow events to be emitted only to the 
clients within the room. Rooms are perfect for creating individual games using 
one server. Without rooms, it would be much more difficult to figure out which 
players and hosts should be connected with each other if multiple people are 
trying to initiate lots of games on the same server.

The following function in [agxgame.js][8] will create a new game room.

    function hostCreateNewGame() {
        // Create a unique Socket.IO Room
        var thisGameId = ( Math.random() * 100000 ) | 0;
    
        // Return the Room ID (gameId) and the socket ID (mySocketId) to the browser client
        this.emit('newGameCreated', {gameId: thisGameId, mySocketId: this.id});
    
        // Join the Room and wait for the players
        this.join(thisGameId.toString());
    };

In the code above, `this` refers to a unique Socket.IO object storing
connection information for the client. The`join` function will place the client
connection into the specified room. In this case the room ID is a randomly 
generated number between 1 and 100000. The room ID is sent back to the client 
and re-used as the unique game ID.

**The client (the Host) detects the `newGameCreated` event from server,
displays the room id as the Game ID on screen.** On the client (in 
[public/app.js][11]), the `App.Host.gameInit` and 
`App.Host.displayNewGameScreen` functions are called in order to display the
randomly generated game ID on screen. The game ID and the Socket.IO room ID are 
the same.

![start_game][12]

### A Player Connects and Clicks Join

**Display the `join-game-template` HTML template.** When a new window is opened
, the client connects via Socket.IO just as before. If the JOIN button is 
clicked, a screen is displayed to collect the player’s name and the desired game
ID.

### The First Player Clicks the Start Button

**The player’s name and game ID get sent to the server.** When a player clicks
‘Start’ after entering his or her name and the appropriate game ID, that 
information is collected into a single*data* object and emitted to the server.
The following code in the`App.Player.onPlayerStartClick` function does this.

    var data = {
        gameId : +($('#inputGameId').val()),
        playerName : $('#inputPlayerName').val() || 'anon'
    };
    IO.socket.emit('playerJoinGame', data);

**On the server, the player is placed into the game room.** When server hears
the`playerJoinGame` event, it calls the `playerJoinGame` function (in 
[agxgame.js][13]). This function places the player’s socket connection into
the room specified by the player. It then re-emits the data (the player’s name 
and socket ID) to the clients in the room so the Host client knows about the 
connected player.

In the code below (from the `playerJoinGame` function), the `gameSocket` refers
to the global Socket.IO object. It allows us to look up all of the created rooms
on the server. If the room is found, the game continues, otherwise an error is 
displayed for the connected player.

    var sock = this;
    
    // Look up the room ID in the Socket.IO manager object.
    var room = gameSocket.manager.rooms["/" + data.gameId];
    
    // If the room exists...
    if( room != undefined ){
        // attach the socket id to the data object.
        data.mySocketId = sock.id;
    
        // Join the room
        sock.join(data.gameId);
    
        // Emit an event notifying the clients that the player has joined the room.
        io.sockets.in(data.gameId).emit('playerJoinedRoom', data);
    
    } else {
        // Otherwise, send an error message back to the player.
        this.emit('error',{message: "This room does not exist."} );
    }

**The player is shown the waiting screen.** When the `playerJoinedRoom` event
is detected by the clients, a message is displayed on screen indicating that the
player joined the game (See screenshot above
).

### Another Player Connects and Clicks Join

**The same sequence occurs until the `playerJoinedRoom` event is emitted from
the server.** When the `playerJoinedRoom` event is detected by the client, and
the client happens to be a Host screen, then`App.Host.updateWaitingScreen` is
called. This function keeps track of the number of players in the room. If there
are two players, then the Host will emit the`hostRoomFull` event.

### The Countdown Begins

**The server detects the `hostRoomFull` event and emits `beginNewGame`.** The
`beginNewGame` event is emitted from the server to the Host and two connected
players in a specified room with the following code:

    function hostPrepareGame(gameId) {
        var sock = this;
        var data = {
            mySocketId : sock.id,
            gameId : gameId
        };
        // Emit the event ONLY to participants in the room   
        io.sockets.in(data.gameId).emit('beginNewGame', data);
    }

Remember, the game ID shown to the players and the Socket.IO room ID where the
client connections are stored*are the same thing*. The `in` function will
target a specified room, and`in(...).emit()` will emit an event only to
participants in that room.

**The Host begins the countdown, and players are informed to “Get Ready”.** On
the client, the`beginNewGame` event is the signal to start the game. The Host
will execute`App.Host.gameCountdown` which loads the `host-game-template` HTML
and displays a five second countdown. On the players’ screen,
`App.Player.gameCountdown` simply displays ‘Get Ready!’.

![gameplay][14]

### The Game Begins!

**The countdown ends, and the Host notifies the server to start the game.**
Because the Host client cannot directly communicate with the players it must 
signal to the server that the countdown has ended. When the countdown reaches 0,
the Host emits a`hostCountdownFinished` event. The server detects it, and
starts the game by calling the`sendWord` function.

**The `sendWord` function prepares the data for the next round in the game.**
In this case, it is the first round. The`sendWord` function has two parts,
gathering the data for the round, and emitting that data to the Host and players
in the room.

The *getWordData* function actually does the heavy lifting of selecting words
for the round and preparing them to be sent to the clients. Word data for each 
round is stored in the following format:

    {
        "words"  : [ "sale","seal","ales","leas" ],
        "decoys" : [ "lead","lamp","seed","eels","lean","cels","lyse","sloe","tels","self" ]
    }

Before each round, the `words` array is randomly shuffled. The first element is
selected as the word displayed on the Host screen. The second element is 
selected as the correct answer. The`decoys` array is also randomly shuffled,
and the first five elements selected as incorrect answers.

The correct answer is then inserted into a random spot in the list of incorrect
answers. This data is emitted back to the client in the following format (see 
the`getWordData` function in [agxgame.js][15]):

    wordData = {
        round: i,          // The current round in the game.
        word : words[0],   // Displayed Word
        answer : words[1], // Correct Answer
        list : decoys      // Word list for player (decoys and answer)
    };

**The Host displays the word on screen.** The `App.Host.newWord` function will
display the word in big letters on the screen, and store a reference to the 
correct answer and the number indicating the current round.

**The player is shown a list of words.** The `App.Player.newWord` function will
parse the list of decoys (which contains the correct answer, as well) and 
construct an unordered list of button elements to display on the players’ 
screens.

### A Player Taps a Word

**The tapped word is sent to the server, and re-emitted to the Host.** Each of
the word buttons displayed to the players has a click handler that will call the
`onPlayerAnswerClick` function when clicked or tapped. The following code will
collect the necessary data, and emit it to the server:

    var $btn = $(this);      // the tapped button
    var answer = $btn.val(); // The tapped word
    
    // Send the player info and tapped word to the server so
    // the Host can check the answer.
    var data = {
        gameId: App.gameId,       // Also the room ID
        playerId: App.mySocketId,
        answer: answer,
        round: App.currentRound
    }
    IO.socket.emit('playerAnswer',data);

**The `playerAnswer` event is detected by the server, and the answer is emitted
to the Host.** The server emits the `hostCheckAnswer` event and attaches the
player’s answer.

### The Host Verifies the Answer

**The Host checks the player’s answer for the current round.** The 
`App.Host.checkAnswer` function first checks to see if the answer came from the
current round. This helps avoid timing problems where one player taps a correct 
answer, and the second player taps the same answer immediately afterwards. 
Because the first player tapped the correct answer, the round should advance, 
and the second player should not get credit (or a penalty) for the tapped answer.

If the correct answer is tapped, the player’s score is incremented and the 
`hostNextRound` event is emitted. If the incorrect answer is tapped, the player
’s score is decremented, and play continues for the current round.

### The Game Ends When the Wordpool is Exhausted

**The `hostNextRound` function on the server will determine if the game is over
.** If the number of the current round is less than the number of elements in
the`wordPool` array, then the game continues with the next round. If the
current round exceeds the number of words available, then the server will emit 
the`gameOver` event to the clients connected in the room.

**The Host displays the winner.** The `App.Host.endGame` function is called on
the client when the server emits the`gameOver` event. In that function, the
Host checks to see which player has the highest score. The winning player’s name
is then displayed on the screen.

Also, the `App.Host.numPlayersInRoom` variable is reset to 0 (even though
technically the players’ socket connections are still in the actual Socket.IO 
room). This is done to allow players a chance to begin a new game. The
`App.Host.isNewGame` flag is set to true as well to indicate that a new game is
about to begin.

![game_over][16]

### The Players Start Again

**The player taps the *Start Again* button and the client emits the 
`playerRestart` event.** The server will notify the Host that a player wishes
to play again by re-emitting the`playerJoinedRoom` event. The function 
`App.Host.updateWaitingScreen` will check the `App.Host.isNewGame` flag, see
that is is true, and display the`create-game-template` on screen until the next
player joins the game.

When the second player clicks the *Start Again* button, the same sequence
occurs and the game begins anew.

## In Conclusion

This application only touches on a couple of the important concepts within the
Socket.IO library and in building real-time web applications with Node.js. If 
you have never worked with real time web applications, hopefully this has been a
good introduction to get you excited about learning more. The
[Socket.IO website][17] provides a decent amount of documentation, as does the
[Socket.IO wiki][18] on GitHub.

 [1]: http://flippinawesome.org/authors/eric-terpstra
 [2]: https://github.com/ericterpstra/anagrammatix
 [3]: https://github.com/STRML/textFit
 [4]: https://github.com/ftlabs/fastclick
 [5]: img/architecture.png
 [6]: http://socket.io/#how-to-use
 [7]: https://github.com/ericterpstra/anagrammatix/blob/master/index.js
 [8]: https://github.com/ericterpstra/anagrammatix/blob/master/agxgame.js
 [9]: https://github.com/ericterpstra/anagrammatix/blob/master/public/app.js
 [10]: img/chrome_network-640x360.jpg

 [11]: https://github.com/ericterpstra/anagrammatix/blob/master/public/app.js#L16
 [12]: img/start_game.jpg
 [13]: https://github.com/ericterpstra/anagrammatix/blob/master/agxgame.js#L95
 [14]: img/gameplay.jpg
 [15]: https://github.com/ericterpstra/anagrammatix/blob/master/agxgame.js#L171
 [16]: img/game_over.jpg
 [17]: http://socket.io/
 [18]: https://github.com/learnboost/socket.io/wiki