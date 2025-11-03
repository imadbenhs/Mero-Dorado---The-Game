<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <title>El Misterio del Mero Dorado</title>
  <style>
    * { box-sizing: border-box; margin: 0; padding: 0; }
    body {
      background: #0e1a2b;
      color: #f0f8ff;
      font-family: 'Segoe UI', Tahoma, sans-serif;
      display: flex;
      justify-content: center;
      padding: 40px 0;
      min-height: 100vh;
      overflow-y: auto; /* ✅ allows full page scrolling if needed */
      align-items: flex-start; /* so the game starts at top */
    }
    #game {
      width: 90%; max-width: 700px;
      background: rgba(20, 35, 60, 0.95);
      border-radius: 16px; padding: 30px;
      box-shadow: 0 0 20px rgba(0,0,0,0.7);
      position: relative;
      /* was: overflow: hidden; -- REMOVE IT! */
      overflow: visible !important; /* ✅ NO cropping of inner content, important to override any inheritance */
    }
    h2 { text-align: center; margin-bottom: 20px; font-size: 1.8rem; color: #ffdd66; }
    #scene {
      margin-bottom: 20px;
      line-height: 1.5;
      animation: fadeIn 0.8s ease-in;
      word-break: break-word;
      overflow-wrap: break-word;
      min-height: 0; /* just in case */
    }
    #choices button {
      display: block; width: 100%; margin: 8px 0; padding: 12px;
      border: none; border-radius: 8px; background: #336699; color: #fff;
      font-size: 1rem; cursor: pointer; transition: background 0.3s, transform 0.1s;
    }
    #choices button:hover { background: #5a9bd3; }
    #choices button:active { transform: scale(0.98); }
    input[type="text"] {
      width: calc(100% - 22px); padding: 10px; margin-bottom: 12px;
      border-radius: 6px; border: none; font-size: 1rem;
    }
    #inventory {
      position: absolute; bottom: 16px; right: 16px;
      background: rgba(0,0,0,0.5); padding: 8px 12px;
      border-radius: 8px; font-size: 0.9rem;
    }
    @keyframes fadeIn {
      from { opacity: 0; transform: translateY(10px); }
      to { opacity: 1; transform: translateY(0); }
    }
  </style>
</head>
<body>
  <div id="game">
    <h2>LOLI-GAME DE "LA COSA"</h2>
    <div id="scene"></div>
    <div id="choices"></div>
    <div id="inventory"></div>
  </div>

  <script>
    const game = {
      scene: 0,
      inventory: [],
      scenes: [
        {
          text: "Capítulo 1: Estás frente a la puerta 90-92 de la Calle Enric Granados, en la maravillosa ciudad de Barcelona. Tres elementos destacan:",
          choices: [
            { text: "Examinar el buzón", next: 1 },
            { text: "Leer un graffiti", next: 2 },
            { text: "Llamar al timbre", next: 3 }
          ]
        },
        {
          text: "Nota en el buzón: “Para abrir el buzón, piensa como un pez. Un pez tiene 0 dedos, 0 orejas y 1 cola. Escribe el número correcto.”",
          input: true,
          correct: "001",
          success: 4,
          fail: 5
        },
        {
          text: "El grafiti dice: “El pez nada mejor cuando nada tiene”. Suena críptico…",
          choices: [
            { text: "Examinar el buzón", next: 1 },
            { text: "Llamar al timbre", next: 3 }
          ]
        },
        {
          text: "Tocas el timbre, pero nadie responde. Tal vez debas mirar el buzón o el grafiti con más atención.",
          choices: [
            { text: "Examinar el buzón", next: 1 },
            { text: "Leer el grafiti", next: 2 }
          ]
        },
        {
          text: "¡Acertaste! El buzón se abre y encuentras una **llave dorada** y un papel: “Peixateria Frederic – Carrer dels Secrets, nº7”.",
          choices: [ { text: "Guardar llave y seguir", next: 6 } ]
        },
        {
          text: "No es correcto. El buzón emite un extraño sonido Loli-acuático. Inténtalo de nuevo.",
          choices: [ { text: "Reintentar", next: 1 } ]
        },
        {
          text: "Has añadido la llave dorada al inventario. La pista sobre la Peixateria Frederic te guía al siguiente paso.",
          choices: [ { text: "Ir a Peixateria Frederic", next: 7 } ]
        },
        {
          text: "Capítulo 2: En la Peixateria Frederic, Frederic, el marido de la dueña Mercè, te lanza un acertijo: ‘¿Cuál de estos peces no puede nadar?’\n\n1) Loli-sword pescadito\n2)[...]",
          choices: [
            { text: "Loli-wooden pescadito", next: 8 },
            { text: "Loli-globi pescadito", next: 9 },
            { text: "Loli-sword pescadito", next: 9 }
          ]
        },
        {
          text: "¡Correcto! Frederic te entrega una foto de la cámara de seguridad: se ve una silueta con un paraguas rojo.",
          choices: [ { text: "Seguir la silueta", next: 10 } ]
        },
        {
          text: "Incorrecto. Frederic dice: “Piensa fuera del agua (o de la caja, como se diga en tu barrio).”",
          choices: [ { text: "Reintentar", next: 7 } ]
        },
        {
          text: "Capítulo 3: Sigues la silueta. En su mochila ves un diario con contraseña escrita en letras mayúsculas. Parece que el término a introducir está relacionado con el lugar de t[...]",
          input: true,
          correct: "PEIXATERIA",
          success: 11,
          fail: 12
        },
        {
          text: "Bien hecho. El diario revela: el Mero Dorado está en un local oculto bajo la calle.",
          choices: [ { text: "Ir al local secreto", next: 13 } ]
        },
        {
          text: "Fallido. Una pista traviesa: Si la contraseña está relacionada con el lugar de TRABAJO de Frederic, que trabaja en PEIXATERIA FREDERIC, y sabemos que la palabra Frederic NO es [...]",
          choices: [ { text: "Reintentar", next: 10 } ]
        },
        {
          text: "Capítulo 4: Estás frente al acceso secreto. La pista dice:\n“Blub blub blub, por la memoria de Nemo... (en mayúsculas y sin espacios)”.",
          input: true,
          correct: "NIÑOINTERIOR",
          success: 14,
          fail: 15
        },
        {
          text: "Acceso concedido. Dentro ves al chef gargantúo con el famoso y legendario Mero Dorado...",
          choices: [ { text: "Confrontar al chef", next: 16 } ]
        },
        {
          text: "No funciona. Acaso alguien ya no se acuerda de un hechizo tan icónico para despertar a tu NIÑO INTERIOR?",
          choices: [ { text: "Reintentar", next: 13 } ]
        },
        {
          text: "Capítulo 5: Confrontación final...",
          choices: [
            { text: "Llamas a la policía", next: 17 },
            { text: "Te comes el pastel de calabaza", next: 18 }
          ]
        },
        {
          text: "Final 1: Entregas al chef. ¡Feliz 4 de noviembre!",
          choices: []
        },
        {
          text: "Final 2: Te comes el pastel. ¡Feliz 4 de noviembre!",
          choices: []
        }
      ]
    };

    const sceneDiv = document.getElementById("scene");
    const choicesDiv = document.getElementById("choices");
    const inventoryDiv = document.getElementById("inventory");

    function renderScene() {
      const cur = game.scenes[game.scene];
      sceneDiv.innerHTML = cur.text;
      choicesDiv.innerHTML = "";
      inventoryDiv.innerHTML = "Inventario: " + (game.inventory.length ? game.inventory.join(", ") : "Vacío");

      if (cur.input) {
        const inp = document.createElement("input");
        inp.type = "text";
        inp.placeholder = "Escribe aquí...";
        choicesDiv.appendChild(inp);

        const btn = document.createElement("button");
        btn.textContent = "Enviar";
        btn.onclick = () => {
          if (inp.value.trim().toUpperCase() === cur.correct) {
            game.scene = cur.success;
          } else {
            game.scene = cur.fail;
          }
          renderScene();
        };
        choicesDiv.appendChild(btn);
      } else {
        cur.choices.forEach(ch => {
          const btn = document.createElement("button");
          btn.textContent = ch.text;
          btn.onclick = () => {
            if (ch.text.includes("Guardar llave")) game.inventory.push("Llave dorada");
            game.scene = ch.next;
            renderScene();
          };
          choicesDiv.appendChild(btn);
        });
      }
    }

    renderScene();
  </script>
</body>
</html>
