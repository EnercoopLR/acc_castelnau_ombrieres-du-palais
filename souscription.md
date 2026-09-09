---
title: "Souscription"
order: 6
in_menu: true
---
# JE SOUSCRIS À UNE BOUCLE D'AUTOCONSOMMATION COLLECTIVE DU TERRITOIRE 

***

## À Castelnau-Le-Lez - Les ombrières du Palais
Je me renseigne pour savoir si je suis éligible géographiquement :

<html lang="fr">

<head>
    <meta charset="UTF-8">
    <title>Adresse localisation</title>

    <link rel="stylesheet" href="https://unpkg.com/leaflet/dist/leaflet.css" />

    <style>
        body {
            margin: 0;
            background: #f3f5f7;
            font-family: Arial, Helvetica, sans-serif;
        }

        .container {
            width: 900px;
            margin: 30px auto;
            background: white;
            border-radius: 16px;
            box-shadow: 0 10px 30px rgba(0, 0, 0, .15);
            overflow: hidden;
        }

        .header {
            padding: 20px;
            border-bottom: 1px solid #ddd;
        }

        input {
            width: 100%;
            padding: 14px;
            font-size: 16px;
            border-radius: 8px;
            border: 1px solid #bbb;
            box-sizing: border-box;
        }

        #suggestions {
            position: relative;
        }

        .list {
            position: absolute;
            left: 0;
            right: 0;
            background: white;
            border-radius: 8px;
            box-shadow: 0 5px 15px rgba(0, 0, 0, .2);
            z-index: 1000;
        }

        .item {
            padding: 10px;
            cursor: pointer;
        }

        .item:hover {
            background: #efefef;
        }

        #map {
            height: 500px;
        }

        .result {
            padding: 20px;
            font-size: 18px;
            font-weight: bold;
        }

        .ok {
            color: #0b8d3f;
        }

        .ko {
            color: #d62828;
        }

        .distance {
            margin-top: 10px;
            font-size: 15px;
            font-weight: normal;
        }

        .green-marker {
            filter: hue-rotate(-70deg) saturate(1.2);
        }
    </style>

</head>

<body>
    <div class="container">
        <div class="header">
            <input id="address" placeholder="Saisir une adresse..." autocomplete="off">
            <div id="suggestions"></div>
        </div>
        <div id="map"></div>
        <div class="result" id="result">
            Entrez une adresse.
        </div>
    </div>

    <script src="https://unpkg.com/leaflet/dist/leaflet.js"></script>
    <script>

        //====================================================
        // PARAMETRES
        //====================================================

        const CENTER = {
            lat: 43.63934,
            lon: 3.90812
        };

        const RAYON = 1000; // mètres

        const Coord_Ombrieres = {
            lat_omb: 43.645228,     //degrés décimaux
            long_omb: 3.917201   //degrés décimaux
        }

        //====================================================

        const map = L.map('map').setView([CENTER.lat, CENTER.lon], 14);

        L.tileLayer(
            "https://data.geopf.fr/wmts?" +
            "SERVICE=WMTS&REQUEST=GetTile&VERSION=1.0.0" +
            "&LAYER=GEOGRAPHICALGRIDSYSTEMS.PLANIGNV2" +
            "&STYLE=normal" +
            "&TILEMATRIXSET=PM" +
            "&FORMAT=image/png" +
            "&TILEMATRIX={z}" +
            "&TILEROW={y}" +
            "&TILECOL={x}"
        ).addTo(map);

        const circle = L.circle(
            [CENTER.lat, CENTER.lon],
            {
                radius: RAYON,
                color: "#0077ff",
                fillColor: "#4ea3ff",
                fillOpacity: .15
            }
        ).addTo(map);

        const greenIcon = L.icon({
            iconUrl: 'https://unpkg.com/leaflet@1.9.4/dist/images/marker-icon-2x.png',
            iconRetinaUrl: 'https://unpkg.com/leaflet@1.9.4/dist/images/marker-icon-2x.png',
            iconSize: [25, 41],
            iconAnchor: [12, 41],
            popupAnchor: [1, -34],
            className: 'green-marker'
        });

        // Creation des Options du marqueur "PDL Ombrières"
        var markerOptions = {
            title: "Poste transfo Ombrières du Palais",
            icon: greenIcon
        }

        const PDL_ombrieres = L.marker(
            [Coord_Ombrieres.lat_omb, Coord_Ombrieres.long_omb], markerOptions

        ).addTo(map);

        PDL_ombrieres.bindPopup(`
            <div class="my-popup">
                <strong>Poste de transformation des Ombrières du Palais</strong>
                <br>
                500 kVA
            </div>
        `);

        let addressMarker = null;

        //====================================================
        // Haversine
        //====================================================

        function distance(lat1, lon1, lat2, lon2) {

            const R = 6371000;

            const toRad = d => d * Math.PI / 180;

            const dLat = toRad(lat2 - lat1);
            const dLon = toRad(lon2 - lon1);

            const a =
                Math.sin(dLat / 2) ** 2 +
                Math.cos(toRad(lat1))
                * Math.cos(toRad(lat2))
                * Math.sin(dLon / 2) ** 2;

            return R * 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));

        }

        //====================================================
        // AUTOCOMPLETION
        //====================================================

        const input = document.getElementById("address");
        const suggestions = document.getElementById("suggestions");

        let timeout = null;

        input.addEventListener("input", () => {

            clearTimeout(timeout);

            const txt = input.value.trim();

            if (txt.length < 3) {
                suggestions.innerHTML = "";
                return;
            }

            timeout = setTimeout(loadSuggestions, 250);

        });

        async function loadSuggestions() {

            const txt = input.value.trim();

            const params = new URLSearchParams({
                text: txt,
                lonlat: `${CENTER.lon},${CENTER.lat}`,
                maximumResponses: 5
            });

            const url = `https://data.geopf.fr/geocodage/completion/?${params}`;

            const r = await fetch(url);

            const data = await r.json();

            let html = "<div class='list'>";

            (data.results || []).forEach(item => {

                html += `
<div class="item"
onclick="selectAddress('${item.fulltext.replace(/'/g, "\\'")}')">
${item.fulltext}
</div>`;

            });

            html += "</div>";

            suggestions.innerHTML = html;

        }

        //====================================================
        // GEOCODAGE
        //====================================================

        async function selectAddress(addr) {

            input.value = addr;
            suggestions.innerHTML = "";

            const url =
                `https://data.geopf.fr/geocodage/search?q=${encodeURIComponent(addr)}`;

            const r = await fetch(url);

            const geo = await r.json();

            if (!geo.features.length)
                return;

            const coords = geo.features[0].geometry.coordinates;

            const lon = coords[0];
            const lat = coords[1];

            if (addressMarker)
                map.removeLayer(addressMarker);

            addressMarker = L.marker([lat, lon]).addTo(map);

            const d = distance(
                CENTER.lat,
                CENTER.lon,
                lat,
                lon
            );

            if (d < 150) {
                map.setView([lat, lon], 18);
            }
            else if (d < 500) {
                map.setView([lat, lon], 17);
            }
            else if (d < 1000) {
                map.setView([lat, lon], 16);
            }
            else {
                map.fitBounds([
                    [CENTER.lat, CENTER.lon],
                    [lat, lon]
                ], {
                    padding: [60, 60]
                });
            }

            const result = document.getElementById("result");

            if (d < RAYON) {

                result.innerHTML =
                    `<span class="ok">
✔ Adresse dans le cercle
</span>
<div class="distance">
Distance : ${Math.round(d)} m
</div>`;

            }
            else {

                result.innerHTML =
                    `<span class="ko">
✖ Adresse hors du cercle
</span>
<div class="distance">
Distance : ${Math.round(d)} m
</div>`;

            }

        }

    </script>
</body>

</html>

<a href="https://form.enercoop.org/autoconsommation-collective-a-castelnau-le-lez" class="bouton">Je rejoins la communauté énergétique</a> 

***

## À Villeneuve-lès-Maguelone
Je me renseigne pour savoir si je suis éligible géographiquement :

<a href="#" class="bouton">Vérifier avec mon adresse postale</a> 