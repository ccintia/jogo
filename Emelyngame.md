<!doctype html>

<html><head>
        <title>GAME-UP</title>
        <!--puxando aquivo separado js-->
        <script src="sprite.js"></script>
        <meta charset = "utf-8"/>
        <style>
            h4{
                color: WHITE;
               	font-family: ARIAL;
               	font-size: 17px;



            }
            CENTER{
               	color: WHITE;
               	font-family: ARIAL;
               	font-size: 40PX;
            }
            canvas{
                position: absolute;
                top: 0px;
                bottom: 0px;
                left: 0px;
                right: 0px;
                margin: auto;
            }  </style>
    </head>
    <body background="bg.png">
        <audio autoplay="autoplay"> <source src="song.mp3" type="audio/mp3" /> </audio>
    <CENTER> GAME-UP: Modo Dificio</CENTER><br><br><br><br><br><br><br><br>
    <br><br><br><br><br><br><br><br><br><br><br><br><br><br>
    <br><br><br><br><br><br><br><br>
    <h4>Por: Emelyn Freire ** 2015 ** GAME-UP                           <br>Música:Rihanna - Bitch Better Have My Money (Official Instrumental)</h4>


    <script> //variaveis de jogo
        //canvas;para desenhar as coisas 
        //frames - associado ao tempo para a ação +1
        var canvas, contexto, ALTURA, LARGURA, frames = 0, maxPulos = 2, velocidade = 15,
                estadoAtual, record, img,
                //config estados
                estados = {
                    jogar: 0,
                    jogando: 1,
                    perdeu: 2
                },
        //config do chao
        chao = {
            y: 550,
            ALTURA: 50,
            cor: "brown",
            desenha: function () {
                contexto.fillStyle = this.cor;
                contexto.fillRect(0, this.y, LARGURA, this.ALTURA);
            }
        },
        //variavel de bloco

        bloco = {
            x: 50,
            y: 0,
            altura: spriteBoneco.altura,
            largura: spriteBoneco.largura,
            cor: "white",
            gravidade: 1.0,
            velocidade: 0,
            forcaDoPulo: 22,
            qntPulos: 0,
            score: 0,
            atualiza: function () {
                this.velocidade += this.gravidade;

                this.y += this.velocidade;

                //para colocar o bloco no jogo e paralo no chao

                if (this.y > chao.y - this.altura) {

                    this.y = chao.y - this.altura;

                    this.qntPulos = 0;
                    this.velocidade = 0;
                }
            },
            //ao terminar o jogo a função é associada
            reset: function () {
                this.velocidade = 0;
                this.y = 0;

                if (this.score > record) {
                    localStorage.setItem("record", this.score);
                    record = this.score;
                }
                this.score = 0;
            },
            pula: function () {
                if (this.qntPulos < maxPulos) {
                    this.velocidade = -this.forcaDoPulo;
                    this.qntPulos++;
                }
            },
            desenha: function () {
                //contexto.fillStyle = this.cor;
                //contexto.fillRect(this.x, this.y, this.largura, this.altura);
                spriteBoneco.desenha(this.x, this.y);
            }
        },
        //variavel de obstaculos
        obstaculos = {
            _obs: [],
            cores: ["#8A2BE2", "#4B0082", "#BA55D3", "#800080", "#FF00FF"],
            tempoInsere: 0,
            insere: function () {
                this._obs.push({
                    x: LARGURA,
                    //largura: 25 + Math.floor(21 * Math.random()),
                    largura: 50,
                    altura: 30 + Math.floor(60 * Math.random()),
                    cor: this.cores[Math.floor(5 * Math.random())]});

                this.tempoInsere = 30 + Math.floor(23 * Math.random());
            },
            atualiza: function () {

                if (this.tempoInsere == 0)
                    this.insere();
                else
                    this.tempoInsere--;

                for (var i = 0, tam = this._obs.length; i < tam; i++) {
                    var obs = this._obs[i];

                    obs.x -= velocidade;
                    if (bloco.x < obs.x + obs.largura && bloco.x + bloco.largura >=
                            obs.x && bloco.y + bloco.altura >= chao.y - obs.altura)
                        estadoAtual = estados.perdeu;
                    else if (obs.x == 0)
                        bloco.score++;
                    else if (obs.x <= -obs.largura) {
                        this._obs.splice(i, 1);
                        tam--;
                        i--;
                    }
                }
            },
            limpa: function () {
                //limpa o vetor
                this._obs = [];
            },
            desenha: function () {
                for (var i = 0, tam = this._obs.length; i < tam; i++) {
                    var obs = this._obs[i];
                    contexto.fillStyle = obs.cor;
                    contexto.fillRect(obs.x, chao.y - obs.altura, obs.largura, obs.altura);
                }
            }};

        //Funçoes principais para uma engine
        function clique(event) {

            //configurações dos estados/abas
            if (estadoAtual == estados.jogando)
                bloco.pula();

            else if (estadoAtual == estados.jogar) {
                estadoAtual = estados.jogando;
            } else if (estadoAtual == estados.perdeu) {
                estadoAtual = estados.jogar;
                obstaculos.limpa();
                bloco.reset();
            }
        }

        function main() {
            //ADD altura e largura na janela 
            ALTURA = window.innerHeight;
            LARGURA = window.innerWidth;

            if (LARGURA >= 1200) {
                LARGURA = 900;
                ALTURA = 550;
            }
            //parametros
            canvas = document.createElement("canvas");
            canvas.width = LARGURA;
            canvas.height = ALTURA;
            canvas.style.border = "1px solid #000";

            //tudo desenhado no contexto vai ser elementos 2d
            contexto = canvas.getContext("2d");
            //add canvas ao html
            document.body.appendChild(canvas);

            //para clicar - se ocorrer essa função ela executara o CLIQUE
            document.addEventListener("mousedown", clique);

            //primeira tela
            estadoAtual = estados.jogar;
            //comando do record armazenamento
            record = localStorage.getItem("record");
            if (record == null)
                record = 0;

            img = new Image();
            img.src = "imagens/back.png";

            roda();
        }
        function roda() {
            atualiza();
            desenha();
            //deixa atualizando sempre
            window.requestAnimationFrame(roda);
        }
        function atualiza() {
            //sempre somam os frames
            frames++;
            bloco.atualiza();

            if (estadoAtual == estados.jogando)
                obstaculos.atualiza();
        }

        function desenha() {

            //background do jogo
            //contexto.fillStyle = "#C0C0C0";
            //contexto.fillRect(0, 0, LARGURA, ALTURA);
            bg.desenha(0, 0);

            //parte do score na tela
            contexto.fillStyle = "pink";
            contexto.font = "50px Arial";
            contexto.fillText(bloco.score, 30, 60);


            if (estadoAtual == estados.jogar) {
                contexto.fillStyle = "green";
                contexto.fillRect(LARGURA / 2 - 50, ALTURA / 2 - 60, 100, 100);
            } else if (estadoAtual == estados.perdeu) {
                contexto.fillStyle = "brown";
                contexto.fillRect(LARGURA / 2 - 50, ALTURA / 2 - 60, 100, 100);

                contexto.save();
                //config do score ao centro
                contexto.translate(LARGURA / 2, ALTURA / 2);
                contexto.fillStyle = "white";


                //CONFIG mensagens de record
                if (bloco.score > record)
                    contexto.fillText("NOVO RECORD!!", -200, -70);

                else if (record < 10)
                    contexto.fillText("Record Atual:" + record, -99, -70);

                else if (record >= 10 && record < 100)
                    contexto.fillText("Record Atual:" + record, -115, -70);

                else
                    contexto.fillText("Record Atual:" + record, -160, -70);

                if (bloco.score < 10)
                    contexto.fillText(bloco.score, -13, 19);

                else if (bloco.score >= 10 && bloco.score < 100)
                    contexto.fillText(bloco.score, -26, 19);

                else if (bloco.score >= 100 && bloco.score < 1000)
                    contexto.fillText(bloco.score, -39, 19);
                contexto.restore();
            } else if (estadoAtual == estados.jogando) {
                obstaculos.desenha();
                //para que os obstaculos fiquem atras do bloco, eles tem que vim primeiro   

            }
            //variavel chao e metodo desenha
            chao.desenha();
            bloco.desenha();
        }
        //inicializa o jogo
        main();
        alert('Bem-vindo ao Game UP!!');
        alert('Vamos jogar?');
        alert('Para começar o jogo click no quadrado verde!!');
        alert('Bom jogo!!');

    </script>

</body>

</html>
