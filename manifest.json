const express = require("express");
const fetch = require("node-fetch");
const app = express();
app.use(express.json());
app.use(express.static("public")); // serve frontend

// แปลง username -> userId
async function getUserId(username) {
  const res = await fetch("https://users.roblox.com/v1/usernames/users", {
    method: "POST",
    headers: {"Content-Type":"application/json"},
    body: JSON.stringify({usernames:[username]})
  });
  const data = await res.json();
  if(!data.data || data.data.length===0) return null;
  return {id:data.data[0].id, name:data.data[0].name};
}

// API ตรวจสอบ group
app.post("/check", async (req, res) => {
  try {
    let { userInput, groupId } = req.body;
    if(!userInput || !groupId) return res.status(400).json({error:"Missing input"});

    let user;
    if(/^\d+$/.test(userInput)){ // numeric -> UserID
      user = {id: userInput, name: userInput};
    } else { // username -> fetch userId
      user = await getUserId(userInput);
      if(!user) return res.json({error:"Username not found"});
    }

    const api = `https://groups.roblox.com/v2/users/${user.id}/groups/roles`;
    const r = await fetch(api);
    const data = await r.json();

    const group = data.data.find(g => g.group.id == groupId);
    if(!group || !group.memberSince) return res.json({error:"User not in group or join date unavailable"});

    const joinDate = new Date(group.memberSince);
    const now = new Date();
    const diffDays = Math.floor((now - joinDate)/(1000*60*60*24));

    res.json({
      username: user.name,
      joinDate: joinDate.toDateString(),
      daysInGroup: diffDays
    });

  } catch(e){
    console.error(e);
    res.status(500).json({error:"Server error"});
  }
});

app.listen(3000, () => console.log("Server running on http://localhost:3000"));
