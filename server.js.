
const express = require("express");
const http = require("http");
const { Server } = require("socket.io");

const app = express();
const server = http.createServer(app);
const io = new Server(server);

app.use(express.json());
app.get("/health", (req, res) => {
  res.json({ status: "ok", game: "AURORA OVERDRIVE" });
});

const rooms = new Map();

io.on("connection", (socket) => {
  socket.on("create-room", (playerName = "Игрок") => {
    const code = Math.random().toString(36).slice(2, 8).toUpperCase();
    rooms.set(code, { players: [socket.id], started: false });
    socket.join(code);
    socket.emit("room-created", { code, playerName });
  });

  socket.on("join-room", (code) => {
    const room = rooms.get(String(code).toUpperCase());
    if (!room || room.started || room.players.length >= 8) {
      socket.emit("room-error", "Комната недоступна");
      return;
    }
    room.players.push(socket.id);
    socket.join(String(code).toUpperCase());
    io.to(String(code).toUpperCase()).emit("players", room.players.length);
  });

  socket.on("disconnect", () => {
    for (const [code, room] of rooms) {
      room.players = room.players.filter((id) => id !== socket.id);
      if (!room.players.length) rooms.delete(code);
      else io.to(code).emit("players", room.players.length);
    }
  });
});

const port = process.env.PORT || 3000;
server.listen(port, "0.0.0.0", () => {
  console.log("AURORA OVERDRIVE server started on " + port);
});
