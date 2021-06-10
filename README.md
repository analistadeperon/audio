# audio
<!DOCTYPE html>
<html>
<head>
    <title></title>
    <script type="text/javascript" src="audiojs/audio.min.js"></script>
    <style type="text/css">
    audio.hide {/*oculta o player padrao do browser*/
        display: none;
    }
    </style>
</head>
<body>

<audio class="hide" controls="controls"></audio>
<ol id="playlistitems"></ol>

<script type="text/javascript">
    function retrieveData(rss, maxItens) {/*extrai dados do feed*/
        var items = rss.getElementsByTagName("item");
        var current, title;
        var playList = [];
        var j = items.length;

        if (maxItens > 0 && maxItens < j) {
            j = maxItens;
        }

        for (var i = 0; i < j; i++) {
            current = items[i].getElementsByTagName("enclosure");
            title   = items[i].getElementsByTagName("title");

            if (current.length > 0) {
                playList.push({
                    "url": current[0].getAttribute("url"),
                    "title": title[0].textContent
                });
            }
        }

        return playList;
    }

    function audioJS(items, total) {/*cria o player com audio.js*/
        var allAudios, current = 0, audio;
        allAudios = audiojs.createAll({
            "trackEnded": function() {
                current++;
                if (current < total) {
                    audio.load(items[current].getAttribute("data-href"));
                    audio.play();
                }
            }
        });

        audio = allAudios[0];
        audio.load(items[0].getAttribute("data-href"));
        audio.play();

        for (var i = 0, j = items.length; i < j; i++) {
            items[i].onclick = function() {
                audio.load(this.getAttribute("data-href"));
                audio.play();
                return false;
            };
        }
    }

    function init(data, maxItens) {/*cria os links*/
        var playList = retrieveData(data, maxItens);
        var el, li, playListItems = document.getElementById("playlistitems");

        if (playList.length > 0) {
            for (var i = 0, j = playList.length; i < j; i++) {
                li = document.createElement("li");
                el = document.createElement("a");
                el.setAttribute("href", "#");
                el.setAttribute("data-href", playList[i].url);
                el.innerHTML = playList[i].title;

                li.appendChild(el);
                playListItems.appendChild(li);
            }
        }

        if (j > 0) {
            audioJS(playListItems.getElementsByTagName("a"), j);
        }
    }

    var ajax = new XMLHttpRequest();
    var url = "http://imagens.globoradio.globo.com/cbn/podcast/cardapios/tecnologia.xml";

    ajax.open("GET", url, true);
    ajax.onreadystatechange = function () {
        if (ajax.readyState === 4) {
            if (ajax.status === 200) {/*se tudo ok envia o feed pro evento init*/
                init(ajax.responseXML, 5);//O segundo parametro limita pra 5 itens no máximo
            } else {/*se falhar ao baixar o xml entao mostra o codigo do erro*/
                alert("Erro: " + ajax.status);
            }
        }
    };
    ajax.send(null);
</script>
</body>
</html>
