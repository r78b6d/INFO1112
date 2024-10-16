java cINFO1112 - Assignment 2 - Tic-Tac-Toe
Tic-Tac-Toe, also known as Noughts and Crosses, is a simple yet engaging two-player game played on
a 3x3 board. The game is typically played by marking the board spaces with ‘X’ and ‘O’, with players
taking turns. The objective of the game is to be the ffrst to align three of your marks—either horizontally,
vertically, or diagonally. More about it on Wikipedia.
Since you have found it quite boring to play alone, you came up with the amazing idea of building a server
that allows people to connect and play with you online. The server has the following features:
• Users can log in
• Users can create game rooms
• Users can join existing game rooms either as players or as viewers
• Users can view players’ moves as they are played
The speciffcations are divided into 3 main sections, which dictate:
• The protocol used to communicate between the client and the server for running a game.
• Server-speciffc runtime details and implementation
• Client-speciffc runtime details and implementation
This assignment is due on Sunday, 20 Oct 2024 at 23:59, Sydney local time.
1 Getting Started
To assist you in the implementation of the assignment, the game logic and input handling for tic-tac-toe
has been implemented in 2 ffles in the scaffold: game.py, and tictactoe.py. Your task is to extend the
logic contained in these ffles, and create:
• a server program which is responsible for handling many games of tic-tac-toe being run simultaneously,
 and,
• a client program, which interacts with a game server (like the one you implement), and allows an
end-user to play tic-tac-toe over a network connection.
You are encouraged to run the function tic tac toe in game.py to play the game and understand how it
works. Of course, you are free to modify these ffles as much as you wish for your submission.
2 Sequence Diagrams
Throughout these speciffcations, we will be using simple sequence diagrams in order to visually depict how
protocol messages are sent back and forth between the client and the server. The shapes below outline the
sequence diagrams used in the assignment speciffcations:
• Messages from source to recipient are represented as a solid arrow with a solid arrow head. These
will be labelled with the protocol message being sent:
EXAMPLE:MESSAGE −−−−−−−−−−−−−−−−−−−−−−−−−−−−→
• Return messages from recipient to source are represented as a dashed arrow with a point arrowhead.
 This is for cases where a recipient is expected to send a message in response back to the source.
They will also be labelled with the protocol message being sent:
EXAMPLE:MESSAGE:RESPONSE
←−−−−−−−−−−−−−−−−−−−−
• Participants, which in this case, will either be a client or a server, can send and receive messages.
These will be depicted as rectangles with an associated lifeline – a dashed vertical line representing
the lifetime of the participant:
Participant
13 Protocol
In order to support playing the game over the network, your programs will need to implement a custom
application-layer protocol as described here in order to facilitate client and server interactions.
All communications will use TCP, and you should use sockets to communicate over the network. More
details about sockets are available from the documentation of Python’s socket module. A how-to guide
for getting started with using TCP sockets is available on Python’s website, and some material will be
included in the lab content to help explain how to create and interact with TCP sockets.
A couple of general notes about the protocol:
• Like many application-layer protocols (e.g. HTTP, SMTP), the protocol is text based, and speciffcally,
uses ASCII encoding for all data.
• When sending data over the network, you will need to ensure you encode data into bytes to send
it over a network socket, and when receiving data, you will need to ensure you decode the received
data to interpret this as a string.
– Hint 1: use the str.encode method to encode a string into bytes with a speciffed encoding.
– Hint 2: use the bytes.decode method to decode bytes to a string.
• You may assume that no message will ever exceed a buffer size of 8192 bytes.
• The majority of protocol messages require the user to be authenticated (after a LOGIN message)
in order for a client to be able to successfully interact with the server. If a category of messages
(one of the subheadings below) requires authentication, the subheading will have (authentication
required) written next to it to indicate this.
– Attempting to send a message from a client requiring authentication without being logged in
will result in a BADAUTH message being sent from the server to the client in response to the sent
message (see below).
3.1 Authentication-related Messages
3.1.1 LOGIN::
This message is sent from the client to the server, and is used to authenticate a user account in order to
be able to play and/or watch games hosted by the server.
The client will send over the username of the user who is attempting to authenticate themselves, and a
plaintext password.
When the server receives this message, it will inspect the user database to check if the user with the given
 exists, and if so, checks that the sent  matches the password hash of the user
in the user database, which has been encrypted with the bcrypt algorithm (see Allowed Modules for
installing the bcrypt Python library on your local device).
• If the username exists in the user database, and the password provided matches the password hash
in the user database, the user will be considered to be successfully authenticated, meaning:
– the server should associate the client socket object of the user which sent the message with
the user account as actively authenticated, meaning the user has permission to interact with all
messages requiring authentication.
∗ multiple clients logging in to the same user account will not be assessed – you are free to
handle this how you see fft.
– the server should respond to the client with a LOGIN:ACKSTATUS:0 message, indicating a successful
 authentication.
– the client should print the message Welcome  to stdout after having received the
above message from the server.
• If the username cannot be found in the user database:
– the server should respond to the client with a LOGIN:ACKSTATUS:1 message.
– the client should print Error: User  not found message to stderr after having
received the message above
• If the username was found in the database, but the password sent did not match the password hash
in the database:
– the server should respond to the client with a LOGIN:ACKSTATUS:2 message.
2– the client should print Error: Wrong password for user  to stderr after having
 received the above message from the server.
• If a LOGIN message was sent with an invalid format, i.e. 0 or 1 arguments:
– the server should respond to the client with a LOGIN:ACKSTATUS:3 message.
– NOTE: Since your client program should be correct, it should never send an invalid LOGIN
message like this, and so there is no error message the client should print in this case. However,
this error message is included to help ensure robustness for the server handling such messages,
such as if you were to use a program such as netcat/nc to directly send messages to the server.
Sequence Diagram
LOGIN::
LOGIN:ACKSTATUS:
Client Server
3.1.2 REGISTER::
This message is sent from the client to the server, and is used to create a new user account in order to be
able to play and view games.
The client will send over the username and password of the user who is attempting to register themselves.
When the server receives this message, it will inspect the user database to ensure the user with the given
 exists does not exist, and will then perform an appropriate action described below:
• If the username does not exist in the user database, the user may be successfully created, meaning:
– the server should create a new user record in the user database, containing:
∗ the username of the new user.
∗ a password hash for the password for the new user account, hashed using the bcrypt
algorithm.
– once the user record is created, the server is required to immediately write to and update
the user database ffle with the new information for the user.
∗ you should either open and close the ffle for writing to do this, or altenatively, call the flush
method to immediately write the contents of the ffle to the disk – either is acceptable, as
long as no data is lost.
– the server should respond to the client with a REGISTER:ACKSTATUS:0 message, indicating a
successful user account creation.
– the client should print the message Successfully created user account  to
stdout after having received the above message from the server.
– NOTE: the user will not be logged in immediately after registration – the client needs to send
a separate LOGIN message to authenticate the user.
• If the username already exists in the user database:
– the server should respond to the client with a REGISTER:ACKSTATUS:1 message, indicating the
user already exists in the user database.
– the client should print the message Error: User  already exists to stderr
after having received the above message from the server.
• If a REGISTER message was sent with an invalid format, i.e. 0 or 1 arguments:
– the server should respond to the client with a REGISTER:ACKSTATUS:2 message.
– NOTE: Since your client program should be correct, it should never send an invalid REGISTER
message like this, and so there is no error message the client should print in this case. However,
this error message is included to help ensure robustness for the server handling such messages,
such as if you were to use a program such as netcat/nc to directly send messages to the server.
Sequence Diagram
3.1.3 BADAUTH
This is a special message which will be sent from the server to the client in response to any protocol
message which requires authentication to perform – so make sure you check for this message when sending
3REGISTER::
REGISTER:ACKSTATUS:
Client Server
an authenticated message from the client!
When a client receives this message, it should output Error: You must be logged in to perform
this action to stderr.
Sequence Diagram

BADAUTH
Client Server
3.2 Room-related Messages (authentication required)
3.2.1 ROOMLIST:
This message is sent from the client to the server, and is used to list rooms that are available to be joined
in the speciffed .
 may be either:
• PLAYER – indicating rooms available to join as a player (and be able to play the game against an
opponent).
• VIEWER – indicating rooms available to join as a viewer (and be able to watch a given game).
When the server receives this message, if  is formatted correctly, the server will respond with the
message ROOMLIST:ACKSTATUS:0:, where  is a list of comma-separated room
names that are available. For instance, if the available room names are:
• Room 1
• epic room 2
• another epic room
 will be formatted as Room 1,epic room 2,another epic room, and thus, the acknowledgement
 message sent from the server back to the client in full will be:
ROOMLIST:ACKSTATUS:0:Room 1,epic room 2,another epic room
Upon receiving the server feedback, the client will output ”Room available to join as : ” which in the example will be ”Room available to join as : Room 1,epic room
2,another epic room”
Error handling
• If a ROOMLIST message was sent with an invalid format, such as having more/less than 1 argument,
or  not being PLAYER or VIEWER:
– the server should respond to the client with a ROOMLIST:ACKSTATUS:1 message.
– the client should rasie the error to stderr”Error: Please input a valid mode.”
Sequence Diagram
ROOMLIST:
ROOMLIST:ACKSTATUS:[:]
Client Server
43.2.2 CREATE:
This message is sent from the client to the server, and is used to create a room with a speciffed .
Room names must meet the following criteria:
• they may only contain alphanumeric characters (a-z, A-Z, 0-9), dashes, spaces, and underscores.
• they must be a maximum of 20 characters in length.
which is validated on the server side.
The server also only allows a maximum of 256 rooms to be created. Once a game is complete, the room
is deleted.
When the server receives this message, it will perform an appropriate action described below:
• If the room named  does not already exist, and the  is valid (from the
above criteria):
– the user that created the room will automatically join the room
– the server should respond to the client with a CREATE:ACKSTATUS:0 message.
– the client should print the message Successfully created room  to stdout
after having received the above message from the server.
– And the client end will wait until there’s another player joined the room. During
the wait, the client isn’t supposed to do anything else other than printing ”Waiting for other
player...”.
∗ Once the client end of the ffrst player in teh room received the BEGIN messsage mentioned
below, the game will start.
• If the room name character is invalid,
– the server should respond to the client with a CREATE:ACKSTATUS:1 message.
– the client should print the message Error: Room  is invalid to stderr after
having received the above message from the server.
• If the  received already refers to a created room:
– the server should respond to the client with a CREATE:ACKSTATUS:2 message.
– the client should print the message Error: Room  already exists to stderr
after having received the above message from the server.
• If there are already 256 rooms created in the server (meaning no further rooms can be created):
– the server should respond to the client with a CREATE:ACKSTATUS:3 message.
– the client should print the message Error: Server already contains a maximum of 256
rooms to stderr after having received the above message from the server.
• If a CREATE message was sent with an invalid format, i.e. 0 or 1 arguments:
– the server should respond to the client with a CREATE:ACKSTATUS:4 message.
– NOTE: Since your client program should be correct, it should never send an invalid CREATE
message like this, and so there is no error message the client should print in this case. However,
this error message is included to help ensure robustness for the server handling such messages,
such as if you were to use a program such as netcat/nc to directly send messages to the server.
Sequence Diagram
CREATE:
CREATE:ACKSTATUS:
Client Server
53.2.3 JOIN::
This message is sent from the client to the server, and is used to join a room with a specified 
in the specified .
 may be either:
• PLAYER – indicating rooms available to join as a player (and be able to play the game against an
opponent).
– NOTE: There may only be 2 players in 1 given room. It is invalid for any further players to
try to join a room as a player in this case.
• VIEWER – indicating rooms available to join as a viewer (and be able to watch a given game).
When the server receives this message, it代 写INFO1112、Python
代做程序编程语言 will perform an appropriate action described below:
• If the room named  exists, and the  provided is a valid mode, and the user can
successfully join the room (from the above criteria):
– the server should add the user into the room in the mode specified by the message sent by the
client.
– the server should respond to the client with a JOIN:ACKSTATUS:0 message.
– the client should print the message Successfully joined room  as a  to stdout after having received the above message from the server, where  is either player or viewer, based on what was sent.
• If there is no room named  in the server:
– the server should respond to the client with a JOIN:ACKSTATUS:1 message.
– the client should print the message Error: No room named  to sderr after having
received the above message from the server.
• If the player is attempting to join  as a PLAYER, but the room is already full (has 2
players):
– the server should respond to the client with a JOIN:ACKSTATUS:2 message.
– the client should print the message Error: The room  already has 2 players
to stderr after having received the above message from the server.
• If a JOIN message was sent with an invalid format, such as having more/less than 2 arguments, or
 not being PLAYER or VIEWER:
– the server should respond to the client with a JOIN:ACKSTATUS:3 message.
– NOTE: Since your client program should be correct, it should never send an invalid JOIN
message like this, and so there is no error message the client should print in this case. However,
this error message is included to help ensure robustness for the server handling such messages,
such as if you were to use a program such as netcat/nc to directly send messages to the server.
Sequence Diagram
JOIN::
JOIN:ACKSTATUS:
Client Server
3.3 Game-related messages (authentication required)
These messages describe interactions between an active game of tic-tac-toe between 2 players.
For any messages which a client sends to the server, the client is required to be authenticated, otherwise,
a BADAUTH message is sent in response by the server.
For these messages, both client and server programs may assume that messages will always be sent and
received in a valid format, hence there is no need to check that messages constructed are of a valid format
in for the below messages.
If, however, a client sends a message specified in this category, but they are not currently in a room (as
either a player or a viewer), the server should respond with a NOROOM message (see below).
63.3.1 BEGIN::
This message is sent from the server to a client, used to inform clients about the players which will be
versing each other when commencing a game of tic-tac-toe.
 is the username of the player who places the first marker (an ’X’), while  is the
username of the player who places the second marker (an ’‘’O’).
When a second player joins a room, the server will send this message to all players and viewers in the
current room, informing them that a game is to begin:
• the client who is logged in as player  will be prompted to begin the game and place their
first marker.
• the client who is logged in as player  will be prompted to wait until  has
placed their first marker (which means that they need to wait until a BOARDSTATUS message has been
sent by the server (see below)).
• any room member’s client end should output the message ”match between  and  will commence, it is currently ’s turn.’”
This message does not require the client to send anything in response.
Sequence Diagram
BEGIN::
Client Server
3.3.2 INPROGRESS::
This message is sent from the server to a client who joins as a viewer to an in-progress game, informing
them:
• the username of the player who’s turn it currently is ().
• the username of the opposing player who is awaiting their turn ().
When the viewing client receives this message, it should output the message ”Match between  and  is currently in progress, it is ’s turn”
This message does not require the client to send anything in response and players’ client ends are not
supposed to receive this message.
Sequence Diagram
INPROGRESS::
Client Server
3.3.3 BOARDSTATUS:
This message is sent from the server to a client to inform them of the current status of the game board
after a move has been made by a player.
Every player and viewer in a room must receive this message after a move has been made.
 explainer
 is a 9-character text string which represents the current state of the 3x3 tic-tac-toe board.
There are 3 characters used to represent a space on the tic-tac-toe board:
• 0 – empty space
• 1 – ’X’ marker
7• 2 – ’O’ marker
The string is essentially a 1D mapping of the 2D array for tic-tac-toe: i.e. each space in the tic-tac-toe
board numbered like so:
1 2 3
4 5 6
7 8 9
is mapped to the  string 123456789.
For instance, a board of the current status:
X O
X
O
would have  equal to 012010020.
Client actions
Any time a client receives this message, they should print the board’s current status to stdout.
Depending on whether the client in the game is a player or viewer, other specific actions will be performed:
• if the client is a player, and:
– has just placed their own marker (having just sent PLACE:: to the server), the client will
recognise that it is the opposing player’s turn (who’s username is known from the BEGIN message
sent when the game has started), and will output that ”It is the opposing player’s turn”,
and wait for the opposing player to complete their turn.
– has been waiting for an opposing player – if this message is received by the player, it means
that the opposing player has placed their marker, and hence, the client should output ”‘It is
the current player’s turn’”, and ask the user to input the x-y coordinates of the marker they
wish to place.
• if the client is a viewer, after having printed the board to stdout, the client should print that it
is the next corresponding player’s turn (the client will have the names of all players from either a
BEGIN or INPROGRESS message sent prior).
This message does not require the client to send anything in response.
Sequence Diagram
BOARDSTATUS:
Client Server
3.3.4 PLACE::
This message is sent from the client to the server, and is used by a player to place a marker on the board
when it is the player’s current turn.
 and  refer to the coordinates on where the marker should be placed on the game board, using
0-based indexing for the spot.
The server makes the assumption that the client sending this message has already validated that  and
 is valid on the client side (i.e. valid coordinates, not currently occupied, etc.), and that the PLACE
message will always contain 2 arguments, and hence will simply process the request as is without needing
to do any error handling. (As a rationale for this, imagine that this protocol will only ever by used by our
own program implementations)
Once a client has sent a PLACE message to the server, the server will place the marker corresponding to the
player appropriately, and send either a BOARDSTATUS message informing the player of the current board’s
status as an acknowledgement, or a GAMEEND message (see below), indicating the game has finished.
8Sequence Diagram
PLACE::

Client Server
3.3.5 FORFEIT
This message is sent from the client to the server, and is used by a player when it is their current turn to
indicate that they wish to forfeit the current game.
Once this message is sent, the game will end, and the server will send a GAMEEND message with a  of 2 (indicating a forfeitted win) for the opposing player (see below) to all players and viewers.
Sequence Diagram
FORFEIT
GAMEEND::2:
Client Server
3.3.6 GAMEEND::[:]
This message is sent from the server to a client to inform them that the game of tic-tac-toe in the current
room has concluded.
 represents the status code of the board at the time of the game ending, in the same
format as described in the BOARDSTATUS message (see above).
The  dictates whether the game has been won by a given player (which would be provided
as an , or whether the game has ended in a draw):
• a  of 0 indicates a player has won the game. In this case, the GAMEEND message will
include a 3rd argument, which is the username of the winner of the game:
GAMEEND::0:
• a  of 1, indicating the game has ended in a draw. No additional arguments will be
provided:
GAMEEND::1
• a  of 2, indicating that a player has forfeited the game. This may result in 3 circumstances:
–
a player has purposely sent a FORFEIT message (see above) to the server when it is their turn.
– a player has disconnected while a game is in progress. In this case, the server will assume that
the player which has disconnected has implicitly forfeitted the game.
– a player’s client end encountered EOF while in the middle of the game (they should disconnect
from the server as well).
In this case, the GAMEEND message will include a 3rd argument, which is the username of the (default)
winner of the game:
GAMEEND::2:
After sending this message, the server will delete the room the game has been occurring in, and allow users
to create/join rooms again.
When clients receives this message, they should print the status of the game’s board, and respond according
to their mode in the room they are currently in:
• if they are a player, and the  was 0 (indicating a win):
9– their username matches the : a message should be printed to stdout,
congratulating the user that they have won the game: ”Congratulations, you won!”
– their username does not match the : a message should be printed to stdout,
informing the user has lost the game, and wishing them better luck next time: ”Sorry you
lost. Good luck next time.”
• if they are a viewer, and the  was 0 (indicating a win), a message should be printed
indicating that ” has won this game”.
• if they are either a player or a viewer, and the  was 1 (indicating a draw), each client
should print a message to stdout informing them that ”Game ended in a draw”.
• if they are either a player or a viewer, and the  was 2 (indicating a player has
forfeitted), each client (if they are still running) should print a message to stdout informing them
” won due to the opposing player forfeiting”.
This message does not require the client to send anything in response.
Sequence Diagram
GAMEEND::[:]
Client Server
3.3.7 NOROOM
This message is sent from the server to a client, in the case that the client which sent a game-related
message (as those described above that can be sent from a client to the server) is, for whatever reason,
not currently inside a room (as a player or a viewer).
Sequence Diagram

NOROOM
Client Server
3.4 Example Game (after joining room): Sequence Diagram
To help illustrate an example of a complete game (from a client being a player’s perspective), a sequence
diagram is included below which helps to illustrate the communications which would occur between a
client and server from the beginning until the end of a game. In this case, the player’s username is alice.
BEGIN:alice:bob
PLACE:1:1
BOARDSTATUS:000010000
BOARDSTATUS:200010000
PLACE:0:2
BOARDSTATUS:200010100
BOARDSTATUS:220010100
PLACE:2:0
GAMEEND:221010100:0:alice
Client Server
• As you can see in this diagram, once the game is started, the server will broadcast 2 players to
everyone in the room. In this case, both viewers and players will know who’s in the game as players.
As mentioned before, the message send from server to client at the start indicates it’s Alice’s turn.
10• The server will wait for Alice to place a piece. If this time Bob is trying to place a piece, we we
mentioend before, Bob’s client is supposed to raise ”It is the opposing player’s turn”.
• After Alice has placed a piece, the server will broadcast the BOARDSTATUS to all of the clients in the
room, which is both players and viewer as mentioned before in client action under BOARDSTATUS
protocol.
– Any time a client receives BORDSTATUS message, they should print the board’s current status
to stdout. Depending on whether the client in the game is a player or viewer, other specific
actions will be performed:
∗ if the client is a player, and:
· has just placed their own marker (having just sent PLACE:: to the server), the
client will recognise that it is the opposing player’s turn (who’s username is known from
the BEGIN message sent when the game has started), and will output that ”It is the
opposing player’s turn”, and wait for the opposing player to complete their turn.
· has been waiting for an opposing player – if this message is received by the player, it
means that the opposing player has placed their marker, and hence, the client should
output ”‘It is the current player’s turn’”, and ask the user to input the x-y coordinates
of the marker they wish to place.
∗ if the client is a viewer, after having printed the board to stdout, the client should print
that it is the next corresponding player’s turn (the client will have the names of all players
from either a BEGIN or INPROGRESS message sent prior).
• And after this, it will be Bob’s turn.
• The rest of the game will follow such behaviour until one of them wins or they reached a draw. And
the client ends are suppposed to print the messages as specified in GAMEEND protocol.
4 Server Program Details
Your server is open to different configurations and is held locally. It needs configurations for which port to
listen to, which database it’s reading from and manage all of the concurrent conenctions. All of the mssages
received from client and server sends are treated as strings. To ensure asynchronous communication, the
server should utilise non-blocking IO and/or processes. Some example methodologies to implement this
include:
• the high-level selectors module, which wraps the lower-level select module to provide an easier
interface for dealing with non-blocking I/O. An example of the module being used for non-blocking
I/O is available in the docs page here.
• the lower-level select module, which allows you to handle sockets only when they have data ready
to read from. An example usage of using select.select for this purpose is included below:
def server_loop(server_sock: socket.socket) -> None:
# Assuming `server_sock` has already been set up to be non-blocking
read_socks = {server_sock}
# Do any other necessary setup
...
# Infinite loop for server (for demonstration)
with server_sock:
while True:
select_read_socks, _, select_except_socks = select.select(
read_socks,
(), # empty tuple
(),
)
for sock in select_read_socks:
11if sock is server_sock:
# New client available
client_sock, client_addr = sock.accept()
client_sock.setblocking(False)
read_socks.add(client_sock)
else:
# `sock` is client socket
client_msg = sock.recv(8192)
if not client_msg:
# Empty message (0 bytes) indicates disconnection
read_socks.remove(sock)
sock.close         
加QQ：99515681  WX：codinghelp  Email: 99515681@qq.com
