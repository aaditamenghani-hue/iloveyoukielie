[i_love_you_kiel_bunny_game.html](https://github.com/user-attachments/files/22949871/i_love_you_kiel_bunny_game.html)
<!-- Bunny Sunflower Game for my wife -->
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>I Love You Kiel</title>
  <style>
    body { font-family: Arial, sans-serif; text-align: center; background: #fdf6f0; }
    #gameArea { position: relative; width: 600px; height: 400px; margin: 20px auto; border: 2px solid #ccc; background: #fff; }
    .bunny, .sunflower { position: absolute; width: 50px; height: 50px; cursor: pointer; }
    #message { display: none; font-size: 24px; margin-top: 20px; color: #ff4b6e; }
  </style>
</head>
<body>
  <h1>I Love You Kiel 🐰🌻</h1>
  <div id="gameArea">
    <img src="https://i.imgur.com/6RkXQ.png" class="bunny" id="bunny" style="left: 10px; top: 170px;">
    <img src="https://i.imgur.com/lSun1.png" class="sunflower" style="left: 500px; top: 50px;">
    <img src="https://i.imgur.com/lSun1.png" class="sunflower" style="left: 300px; top: 300px;">
    <img src="https://i.imgur.com/lSun1.png" class="sunflower" style="left: 100px; top: 80px;">
  </div>
  <div id="message">You collected all the sunflowers! 💛 This is for my wife!</div>
  <script>
    const bunny = document.getElementById('bunny');
    const sunflowers = document.querySelectorAll('.sunflower');
    let collected = 0;

    sunflowers.forEach(flower => {
      flower.addEventListener('click', () => {
        flower.style.display = 'none';
        collected++;
        if (collected === 3) {
          document.getElementById('message').style.display = 'block';
        }
      });
    });
  </script>
</body>
</html>
