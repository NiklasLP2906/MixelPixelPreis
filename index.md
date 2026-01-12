<!DOCTYPE html>
<html lang="de">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>MixelPreise | Master Edition</title>
    <style>
        :root {
            --primary: #2563eb; --bg: #0f172a; --card: #1e293b;
            --text: #f8fafc; --muted: #94a3b8; --border: #334155;
            --success: #22c55e; --danger: #ef4444; --accent: #3b82f6;
        }
        * { box-sizing: border-box; }
        html, body { height: 100vh; margin: 0; overflow: hidden; font-family: 'Inter', sans-serif; background: var(--bg); color: var(--text); }
        body { display: flex; flex-direction: column; }
        .top-bar { background: #020617; padding: 10px 20px; display: flex; justify-content: space-between; align-items: center; border-bottom: 1px solid var(--border); flex-shrink: 0; }
        .main-content { flex: 1; display: flex; flex-direction: column; padding: 20px; gap: 15px; min-height: 0; }
        header { display: flex; justify-content: space-between; align-items: center; flex-shrink: 0; }
        .search-area { display: flex; gap: 10px; }
        .search-area input { background: var(--card); border: 1px solid var(--border); padding: 12px; border-radius: 8px; color: white; width: 300px; outline: none; }
        .table-container { flex: 1; background: var(--card); border: 1px solid var(--border); border-radius: 12px; overflow-y: auto; position: relative; }
        table { width: 100%; border-collapse: collapse; text-align: left; table-layout: fixed; }
        thead th { position: sticky; top: 0; background: #2d3a4f; padding: 15px 20px; font-size: 0.7rem; text-transform: uppercase; color: var(--muted); z-index: 10; border-bottom: 2px solid var(--border); cursor: pointer; }
        td { padding: 12px 20px; border-bottom: 1px solid var(--border); font-size: 0.85rem; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }
        tr:hover { background: #2d3a4f; }
        .w-ev { width: 150px; } .w-n { width: auto; } .w-p { width: 120px; }
        .btn { padding: 8px 16px; border-radius: 6px; cursor: pointer; font-weight: 600; border: none; transition: 0.2s; }
        .btn-p { background: var(--primary); color: white; }
        .btn-d { background: var(--danger); color: white; }
        .btn-s { background: #334155; color: white; font-size: 0.75rem; margin-right: 5px; }
        .btn-add { background: var(--success); color: white; font-size: 0.75rem; margin-right: 5px; }
        .modal { display: none; position: fixed; inset: 0; background: rgba(0,0,0,0.8); backdrop-filter: blur(5px); align-items: center; justify-content: center; z-index: 1000; }
        .modal-content { background: var(--card); padding: 25px; border-radius: 15px; width: 400px; border: 1px solid var(--border); }
        .modal-content input { width: 100%; padding: 12px; margin-bottom: 15px; background: #0f172a; border: 1px solid var(--border); color: white; border-radius: 8px; }
        ::-webkit-scrollbar { width: 8px; } ::-webkit-scrollbar-thumb { background: #334155; border-radius: 4px; }
    </style>
</head>
<body>

<div class="top-bar">
    <div id="adminControls" style="display:none">
        <button class="btn btn-add" onclick="openAddModal()">➕ Item hinzufügen</button>
        <button class="btn btn-s" onclick="exportFullHTML()">💾 HTML speichern</button>
        <button class="btn btn-d" onclick="resetToCode()">⚙️ Reset (Code-Stand)</button>
    </div>
    <div style="flex:1"></div>
    <div id="loginArea">
        <input type="password" id="pw" placeholder="Passwort..." style="background:#1e293b; border:1px solid #334155; color:white; padding:6px; border-radius:6px;">
        <button class="btn btn-p" onclick="login()">Login</button>
    </div>
    <div id="adminArea" style="display:none">
        <span style="color:var(--success); font-weight:bold;">ADMIN</span>
        <button class="btn btn-d" style="margin-left:15px; padding:5px 10px;" onclick="location.reload()">Logout</button>
    </div>
</div>

<div class="main-content">
    <header>
        <h1 style="margin:0;">Mixel<span style="color:var(--accent)">Preise</span></h1>
        <div class="search-area">
            <input type="text" id="searchInput" placeholder="Suchen..." onkeyup="render()">
            <div id="counter" style="background:var(--accent); padding:10px 18px; border-radius:8px; font-weight:bold; min-width:110px; text-align:center;">0 Items</div>
        </div>
    </header>
    <div class="table-container">
        <table>
            <thead>
                <tr>
                    <th class="w-ev" onclick="resort('ev')">Event ↕</th>
                    <th class="w-n" onclick="resort('n')">Gegenstand ↕</th>
                    <th class="w-p" onclick="resort('buy')">Ankauf ↕</th>
                    <th class="w-p" onclick="resort('sell')">Verkauf ↕</th>
                    <th class="w-p" onclick="resort('markt')">Markt ↕</th>
                    <th class="w-p">Profit</th>
                    <th id="adminTh" style="display:none; width:150px;">Aktion</th>
                </tr>
            </thead>
            <tbody id="tableBody"></tbody>
        </table>
    </div>
</div>

<div id="editModal" class="modal">
    <div class="modal-content">
        <h3 id="mTitle" style="margin-top:0; color:var(--accent)">Preis-Update</h3>
        <label>Ankauf ($)</label><input type="number" id="mBuy">
        <label>Verkauf ($)</label><input type="number" id="mSell">
        <label>Marktpreis ($)</label><input type="number" id="mMarkt">
        <button class="btn btn-p" style="width:100%;" onclick="save()">Speichern</button>
        <button class="btn btn-d" style="width:100%; margin-top:10px;" onclick="clearValues()">Werte auf 0 setzen</button>
        <button class="btn" style="width:100%; margin-top:10px;" onclick="closeModal('editModal')">Abbrechen</button>
    </div>
</div>

<div id="addModal" class="modal">
    <div class="modal-content">
        <h3 style="margin-top:0; color:var(--success)">Neues Item hinzufügen</h3>
        <label>Event Name</label><input type="text" id="addEv" placeholder="z.B. Winter 2025">
        <label>Gegenstand</label><input type="text" id="addN" placeholder="Name des Items">
        <button class="btn btn-p" style="width:100%;" onclick="addNewItem()">Hinzufügen</button>
        <button class="btn" style="width:100%; margin-top:10px;" onclick="closeModal('addModal')">Abbrechen</button>
    </div>
</div>

<script id="mainScript">
    const ADMIN_PW = "MixelPreisGooner";
    let isAdmin = false, sCol = 'ev', sAsc = true, editIdx = -1;

    // --- DATENSATZ ---
    const rawData = [
        {ev:"Valentinstag 2024", n:"Armors Pfeil", buy:0, sell:0, markt:0}, {ev:"Valentinstag 2024", n:"Valentina", buy:0, sell:0, markt:0}, {ev:"Valentinstag 2024", n:"Blumenstrauß (Kopf)", buy:0, sell:0, markt:0}, {ev:"Valentinstag 2024", n:"Korb voller Rosen (Kopf)", buy:0, sell:0, markt:0},
        {ev:"Valentinstag 2025", n:"Armors Pfeil", buy:0, sell:0, markt:0}, {ev:"Valentinstag 2025", n:"Valentina", buy:0, sell:0, markt:0}, {ev:"Valentinstag 2025", n:"Blumenstrauß", buy:0, sell:0, markt:0}, {ev:"Valentinstag 2025", n:"Pralinen Schachtel", buy:0, sell:0, markt:0}, {ev:"Valentinstag 2025", n:"Luftkuss Kanone", buy:0, sell:0, markt:0}, {ev:"Valentinstag 2025", n:"Blumenstrauß (Kopf)", buy:0, sell:0, markt:0}, {ev:"Valentinstag 2025", n:"Korb voller Rosen (Kopf)", buy:0, sell:0, markt:0}, {ev:"Valentinstag 2025", n:"Pralinen (Kopf)", buy:0, sell:0, markt:0}, {ev:"Valentinstag 2025", n:"Kaninchen Kuscheltier (Kopf)", buy:0, sell:0, markt:0},
        {ev:"Ostern 2020", n:"Ostern '20 Kopf", buy:0, sell:0, markt:0}, {ev:"Ostern 2020", n:"Ostern '20 Harnisch", buy:0, sell:0, markt:0}, {ev:"Ostern 2020", n:"Ostern '20 Hose", buy:0, sell:0, markt:0}, {ev:"Ostern 2020", n:"Ostern '20 Stiefel", buy:0, sell:0, markt:0}, {ev:"Ostern 2020", n:"Ostern '20 Schwert", buy:0, sell:0, markt:0}, {ev:"Ostern 2020", n:"Ostern '20 Spitzhacke", buy:0, sell:0, markt:0}, {ev:"Ostern 2020", n:"Ostern '20 Axt", buy:0, sell:0, markt:0}, {ev:"Ostern 2020", n:"Ostern '20 Schaufel", buy:0, sell:0, markt:0}, {ev:"Ostern 2020", n:"Ostern '20 Magische Spitzhacke", buy:0, sell:0, markt:0}, {ev:"Ostern 2020", n:"Ostern '20 Regenbogen", buy:0, sell:0, markt:0}, {ev:"Ostern 2020", n:"Ostern '20 Farbkopf", buy:0, sell:0, markt:0}, {ev:"Ostern 2020", n:"Ostern '20 Umhang", buy:0, sell:0, markt:0}, {ev:"Ostern 2020", n:"Ostern '20 Jogginghose", buy:0, sell:0, markt:0}, {ev:"Ostern 2020", n:"Ostern '20 Hasenpfote", buy:0, sell:0, markt:0}, {ev:"Ostern 2020", n:"Ostern '20 Brücken Ei", buy:0, sell:0, markt:0}, {ev:"Ostern 2020", n:"Ostern '20 Eier Kanone", buy:0, sell:0, markt:0}, {ev:"Ostern 2020", n:"Ostern '20 Sprungfeder", buy:0, sell:0, markt:0}, {ev:"Ostern 2020", n:"Ostern '20 Unendliche Rakete", buy:0, sell:0, markt:0}, {ev:"Ostern 2020", n:"Ostern '20 Küken (Kopf)", buy:0, sell:0, markt:0}, {ev:"Ostern 2020", n:"Ostern '20 Korb (Kopf)", buy:0, sell:0, markt:0},
        {ev:"Ostern 2024", n:"Ostern '24 Alfreds Kopf", buy:0, sell:0, markt:0}, {ev:"Ostern 2024", n:"Ostern '24 Alfreds Harnisch", buy:0, sell:0, markt:0}, {ev:"Ostern 2024", n:"Ostern '24 Alfreds Shorts", buy:0, sell:0, markt:0}, {ev:"Ostern 2024", n:"Ostern '24 Alfreds Stiefel", buy:0, sell:0, markt:0}, {ev:"Ostern 2024", n:"Ostern '24 Kükenschwert", buy:0, sell:0, markt:0}, {ev:"Ostern 2024", n:"Ostern '24 Kükenspitzhacke", buy:0, sell:0, markt:0}, {ev:"Ostern 2024", n:"Ostern '24 Kükenaxt", buy:0, sell:0, markt:0}, {ev:"Ostern 2024", n:"Ostern '24 Kükenschaufel", buy:0, sell:0, markt:0}, {ev:"Ostern 2024", n:"Ostern '24 Eier Kanone", buy:0, sell:0, markt:0}, {ev:"Ostern 2024", n:"Ostern '24 Autogramm", buy:0, sell:0, markt:0}, {ev:"Ostern 2024", n:"Floppy", buy:0, sell:0, markt:0}, {ev:"Ostern 2024", n:"Osterkorb", buy:0, sell:0, markt:0}, {ev:"Ostern 2024", n:"Ostern '24 Lilanes Ei", buy:0, sell:0, markt:0}, {ev:"Ostern 2024", n:"Ostern '24 Grünes Ei", buy:0, sell:0, markt:0}, {ev:"Ostern 2024", n:"Ostern '24 Blaues Ei", buy:0, sell:0, markt:0}, {ev:"Ostern 2024", n:"Ostern '24 Braunes Ei", buy:0, sell:0, markt:0}, {ev:"Ostern 2024", n:"Ostern '24 Gelbes Ei", buy:0, sell:0, markt:0},
        {ev:"Jubiläum 2021", n:"Jubiläum '21 Mixel-Kuchen", buy:0, sell:0, markt:0},
        {ev:"Jubiläum 2022", n:"Jubiläum '22 Pinata-Kuchen", buy:0, sell:0, markt:0}, {ev:"Jubiläum 2022", n:"Jubiläum '22 Shulkerkiste", buy:0, sell:0, markt:0},
        {ev:"Jubiläum 2023", n:"Jubiläum '23 Community Cupcake", buy:0, sell:0, markt:0}, {ev:"Jubiläum 2023", n:"Jubiläum '23 Konfetti Kanone", buy:0, sell:0, markt:0}, {ev:"Jubiläum 2023", n:"Uralte verstaubte Kiste mit Inhalt", buy:0, sell:0, markt:0}, {ev:"Jubiläum 2023", n:"Montys Bosfisch-Finder", buy:0, sell:0, markt:0}, {ev:"Jubiläum 2023", n:"Hecamus JnR Trainings-Stiefel", buy:0, sell:0, markt:0}, {ev:"Jubiläum 2023", n:"Kave96s Flachwitzebuch", buy:0, sell:0, markt:0}, {ev:"Jubiläum 2023", n:"Alinas Teetasse", buy:0, sell:0, markt:0}, {ev:"Jubiläum 2023", n:"FlyingFingers Flash-Comics", buy:0, sell:0, markt:0},
        {ev:"Jubiläum 2024", n:"Jubiläum '24 Geburtstagskuchen", buy:0, sell:0, markt:0}, {ev:"Jubiläum 2024", n:"Jubiläum '24 Konfetti Kanone", buy:0, sell:0, markt:0}, {ev:"Jubiläum 2024", n:"Cusco", buy:0, sell:0, markt:0}, {ev:"Jubiläum 2024", n:"Goodies", buy:0, sell:0, markt:0}, {ev:"Jubiläum 2024", n:"Jubiläum '24 Mixel Götter Trunk", buy:0, sell:0, markt:0}, {ev:"Jubiläum 2024", n:"Head-Team Titel auswählen", buy:0, sell:0, markt:0}, {ev:"Jubiläum 2024", n:"Head-Team Emote auswählen", buy:0, sell:0, markt:0},
        {ev:"Jubiläum 2025", n:"Jubiläum '25 Infinitypearl", buy:0, sell:0, markt:0}, {ev:"Jubiläum 2025", n:"Knalltüte", buy:0, sell:0, markt:0}, {ev:"Jubiläum 2025", n:"Jubiläum '25 Alte Hacke", buy:0, sell:0, markt:0}, {ev:"Jubiläum 2025", n:"Jubiläum '25 Zeitmaschiene", buy:0, sell:0, markt:0}, {ev:"Jubiläum 2025", n:"Jubiläum '25 Skin Auswahltruhe", buy:0, sell:0, markt:0}, {ev:"Jubiläum 2025", n:"Jubiläum '25 Pinata To Go", buy:0, sell:0, markt:0}, {ev:"Jubiläum 2025", n:"Kave96's Snack", buy:0, sell:0, markt:0}, {ev:"Jubiläum 2025", n:"FlyingFinger's Energieriegel", buy:0, sell:0, markt:0}, {ev:"Jubiläum 2025", n:"Jubiläum '25 Schere Stein Papier", buy:0, sell:0, markt:0},
        {ev:"Sommer 2021", n:"Sommer '21 Taucherflossen", buy:0, sell:0, markt:0}, {ev:"Sommer 2021", n:"Sommer '21 Wasser Pistole", buy:0, sell:0, markt:0},
        {ev:"Sommer 2023", n:"Sommer '23 Schildkröten-Helm", buy:0, sell:0, markt:0}, {ev:"Sommer 2023", n:"Sommer '23 Schildkröten-Panzer", buy:0, sell:0, markt:0}, {ev:"Sommer 2023", n:"Sommer '23 Schildkröten-Beine", buy:0, sell:0, markt:0}, {ev:"Sommer 2023", n:"Sommer '23 Schildkröten-Flossen", buy:0, sell:0, markt:0}, {ev:"Sommer 2023", n:"Sommer '23 Volleyball Kanone", buy:0, sell:0, markt:0}, {ev:"Sommer 2023", n:"Sommer '23 Poseidons Spitzhacke", buy:0, sell:0, markt:0}, {ev:"Sommer 2023", n:"Sommer '23 Torpedo Elytra", buy:0, sell:0, markt:0}, {ev:"Sommer 2023", n:"Sommer '23 Magischer Dreizack", buy:0, sell:0, markt:0}, {ev:"Sommer 2023", n:"Aquarium", buy:0, sell:0, markt:0}, {ev:"Sommer 2023", n:"Sommer '23 Disco Fisch", buy:0, sell:0, markt:0}, {ev:"Sommer 2023", n:"Sommer '23 Metalldetektor", buy:0, sell:0, markt:0}, {ev:"Sommer 2023", n:"Sommer '23 Cheat-Code", buy:0, sell:0, markt:0}, {ev:"Sommer 2023", n:"Sommer Items", buy:0, sell:0, markt:0}, {ev:"Sommer 2023", n:"Sommer '23 Keramik Kratzer", buy:0, sell:0, markt:0}, {ev:"Sommer 2023", n:"Sommer '23 Mobiler Betonmischer", buy:0, sell:0, markt:0}, {ev:"Sommer 2023", n:"Sommer '23 Heißes Eisen", buy:0, sell:0, markt:0},
        {ev:"Sommer 2024", n:"Sommer '24 Skin Auswahltruhe", buy:0, sell:0, markt:0}, {ev:"Sommer 2024", n:"Sommer '24 Krabben Grammophon", buy:0, sell:0, markt:0}, {ev:"Sommer 2024", n:"Sommer '24 Wasser Pistole V2", buy:0, sell:0, markt:0}, {ev:"Sommer 2024", n:"Sommer '24 Tic-Tac-Toe", buy:0, sell:0, markt:0}, {ev:"Sommer 2024", n:"Sommer '24 BBQ-Schwert", buy:0, sell:0, markt:0}, {ev:"Sommer 2024", n:"Tinkerbells Feenbeutel", buy:0, sell:0, markt:0}, {ev:"Sommer 2024", n:"Sommer Verwandlung auswählen", buy:0, sell:0, markt:0}, {ev:"Sommer 2024", n:"Pua", buy:0, sell:0, markt:0},
        {ev:"Sommer 2025", n:"Sommer '25 Wasserbomben Werfer", buy:0, sell:0, markt:0}, {ev:"Sommer 2025", n:"Sommer '25 Skinauswahltruhe", buy:0, sell:0, markt:0}, {ev:"Sommer 2025", n:"Käseversteck", buy:0, sell:0, markt:0}, {ev:"Sommer 2025", n:"Sommer '25 Woolinator 3000", buy:0, sell:0, markt:0}, {ev:"Sommer 2025", n:"Sommer '25 Disco Axolotl", buy:0, sell:0, markt:0}, {ev:"Sommer 2025", n:"Sommer '25 Teppichmesser", buy:0, sell:0, markt:0}, {ev:"Sommer 2025", n:"Sommer '25 Oxidator", buy:0, sell:0, markt:0}, {ev:"Sommer 2025", n:"Sommer Verwandlung auswählen", buy:0, sell:0, markt:0}, {ev:"Sommer 2025", n:"Sommer '25 Geduldsfaden", buy:0, sell:0, markt:0}, {ev:"Sommer 2025", n:"Sommer '25 Captcha Duell", buy:0, sell:0, markt:0},
        {ev:"Halloween 2019", n:"Halloween '19 Kopf", buy:0, sell:0, markt:0}, {ev:"Halloween 2019", n:"Halloween '19 Harnisch", buy:0, sell:0, markt:0}, {ev:"Halloween 2019", n:"Halloween '19 Hose", buy:0, sell:0, markt:0}, {ev:"Halloween 2019", n:"Halloween '19 Stiefel", buy:0, sell:0, markt:0}, {ev:"Halloween 2019", n:"Halloween '19 Schwert", buy:0, sell:0, markt:0}, {ev:"Halloween 2019", n:"Halloween '19 Spitzhacke", buy:0, sell:0, markt:0}, {ev:"Halloween 2019", n:"Halloween '19 Kampfaxt", buy:0, sell:0, markt:0}, {ev:"Halloween 2019", n:"Halloween '19 Schaufel", buy:0, sell:0, markt:0}, {ev:"Halloween 2019", n:"Halloween '19 Gefäß voller Blut", buy:0, sell:0, markt:0}, {ev:"Halloween 2019", n:"Halloween '19 Kürbislaterne", buy:0, sell:0, markt:0}, {ev:"Halloween 2019", n:"Halloween '19 Satans Buch", buy:0, sell:0, markt:0},
        {ev:"Halloween 2020", n:"Halloween '20 Fledermaus Kanone", buy:0, sell:0, markt:0},
        {ev:"Halloween 2022", n:"Halloween '22 Süßes oder Saures Axt", buy:0, sell:0, markt:0}, {ev:"Halloween 2022", n:"Halloween '22 Horrormaske", buy:0, sell:0, markt:0},
        {ev:"Halloween 2023", n:"Halloween '23 Spinnenschreck", buy:0, sell:0, markt:0}, {ev:"Halloween 2023", n:"Halloween '23 Zuckertüte", buy:0, sell:0, markt:0}, {ev:"Halloween 2023", n:"Halloween '23 Geisterflammen", buy:0, sell:0, markt:0}, {ev:"Halloween 2023", n:"Naschtüte", buy:0, sell:0, markt:0}, {ev:"Halloween 2023", n:"Hektor", buy:0, sell:0, markt:0},
        {ev:"Halloween 2024", n:"Halloween '24 Skin Auswahltruhe", buy:0, sell:0, markt:0}, {ev:"Halloween 2024", n:"Halloween '24 Totenkopfkanone", buy:0, sell:0, markt:0}, {ev:"Halloween 2024", n:"Halloween '24 Friedhofschlüssel", buy:0, sell:0, markt:0}, {ev:"Halloween 2024", n:"Halloween '24 Gruselarmbrust", buy:0, sell:0, markt:0}, {ev:"Halloween 2024", n:"Halloween '24 Totengräberschaufel", buy:0, sell:0, markt:0}, {ev:"Halloween 2024", n:"Halloween '24 Hexenbesen", buy:0, sell:0, markt:0}, {ev:"Halloween 2024", n:"Halloween Verwandlung auswählen", buy:0, sell:0, markt:0}, {ev:"Halloween 2024", n:"Gruselkiste", buy:0, sell:0, markt:0}, {ev:"Halloween 2024", n:"Luna", buy:0, sell:0, markt:0},
        {ev:"Halloween 2025", n:"Halloween '25 Skin Auswahltruhe", buy:0, sell:0, markt:0}, {ev:"Halloween 2025", n:"Halloween '25 Berserker Schwert", buy:0, sell:0, markt:0}, {ev:"Halloween 2025", n:"Halloween '25 Sporenblüten Kanone", buy:0, sell:0, markt:0}, {ev:"Halloween 2025", n:"Halloween '25 Berserker Axt", buy:0, sell:0, markt:0}, {ev:"Halloween 2025", n:"Infernotruhe", buy:0, sell:0, markt:0}, {ev:"Halloween 2025", n:"Halloween '25 Verfluchte Spitzhacke", buy:0, sell:0, markt:0}, {ev:"Halloween 2025", n:"Halloween '25 Seelen Gießkanne", buy:0, sell:0, markt:0}, {ev:"Halloween 2025", n:"Halloween '25 Knarz Bodyguard", buy:0, sell:0, markt:0},
        {ev:"Winter 2019", n:"Winter '19 Eis Klinge", buy:0, sell:0, markt:0}, {ev:"Winter 2019", n:"Winter '19 Brennende Spitzhacke", buy:0, sell:0, markt:0}, {ev:"Winter 2019", n:"Winter '19 Schneeschaufel", buy:0, sell:0, markt:0}, {ev:"Winter 2019", n:"Winter '19 Krampus Dolch", buy:0, sell:0, markt:0}, {ev:"Winter 2019", n:"Winter '19 Langbogen", buy:0, sell:0, markt:0}, {ev:"Winter 2019", n:"Winter '19 Weihnachts Rute", buy:0, sell:0, markt:0}, {ev:"Winter 2019", n:"Winter '19 Schnee Elytra", buy:0, sell:0, markt:0}, {ev:"Winter 2019", n:"Winter '19 Schneehose", buy:0, sell:0, markt:0}, {ev:"Winter 2019", n:"Winter '19 Bodenloses Fass", buy:0, sell:0, markt:0}, {ev:"Winter 2019", n:"Winter '19 Schneeball Kanone", buy:0, sell:0, markt:0}, {ev:"Winter 2019", n:"Weihnachts Feuerwerk 2019", buy:0, sell:0, markt:0}, {ev:"Winter 2019", n:"Weihnachts Spekulatius 2019", buy:0, sell:0, markt:0}, {ev:"Winter 2019", n:"Santa Claus Head 2019", buy:0, sell:0, markt:0}, {ev:"Winter 2019", n:"Winter '19 Grinch Kopf", buy:0, sell:0, markt:0},
        {ev:"Winter 2020", n:"Winter '20 Kopf", buy:0, sell:0, markt:0}, {ev:"Winter 2020", n:"Winter '20 Mantel", buy:0, sell:0, markt:0}, {ev:"Winter 2020", n:"Winter '20 Hose", buy:0, sell:0, markt:0}, {ev:"Winter 2020", n:"Winter '20 Stiefel", buy:0, sell:0, markt:0}, {ev:"Winter 2020", n:"Winter '20 X-Mas Bogen", buy:0, sell:0, markt:0}, {ev:"Winter 2020", n:"Winter '20 Schneeschaufel", buy:0, sell:0, markt:0}, {ev:"Winter 2020", n:"Winter '20 Weihnachts Knaller", buy:0, sell:0, markt:0}, {ev:"Winter 2020", n:"Monty's Futter", buy:0, sell:0, markt:0}, {ev:"Winter 2020", n:"Lars, der kleine Eisbär", buy:0, sell:0, markt:0}, {ev:"Winter 2020", n:"Keksdose", buy:0, sell:0, markt:0}, {ev:"Winter 2020", n:"Winter '20 Nikolaus Kopf", buy:0, sell:0, markt:0},
        {ev:"Winter 2021", n:"Winter '21 Santa's Schwert", buy:0, sell:0, markt:0}, {ev:"Winter 2021", n:"Winter '21 Santa's Spitzhacke", buy:0, sell:0, markt:0}, {ev:"Winter 2021", n:"Winter '21 Santa's Axt", buy:0, sell:0, markt:0}, {ev:"Winter 2021", n:"Winter '21 Santa's Schaufel", buy:0, sell:0, markt:0}, {ev:"Winter 2021", n:"Winter '21 Angel des Weisen", buy:0, sell:0, markt:0}, {ev:"Winter 2021", n:"Winter '21 Santa's Schild", buy:0, sell:0, markt:0}, {ev:"Winter 2021", n:"Winter '21 Engels Flügel", buy:0, sell:0, markt:0}, {ev:"Winter 2021", n:"Winter '21 Schneeball Kanone", buy:0, sell:0, markt:0}, {ev:"Winter 2021", n:"Winter '21 Weihnachts Knaller", buy:0, sell:0, markt:0}, {ev:"Winter 2021", n:"Rudolf's Spielgefährte", buy:0, sell:0, markt:0}, {ev:"Winter 2021", n:"Winter '21 Geschenke-Rucksack", buy:0, sell:0, markt:0}, {ev:"Winter 2021", n:"Winter '21 Mixel Nussknacker", buy:0, sell:0, markt:0}, {ev:"Winter 2021", n:"Winter '21 Backrezepte", buy:0, sell:0, markt:0}, {ev:"Winter 2021", n:"Winter '21 Lebkuchen", buy:0, sell:0, markt:0},
        {ev:"Winter 2022", n:"Winter '22 Kopf", buy:0, sell:0, markt:0}, {ev:"Winter 2022", n:"Winter '22 Pelz", buy:0, sell:0, markt:0}, {ev:"Winter 2022", n:"Winter '22 Hose", buy:0, sell:0, markt:0}, {ev:"Winter 2022", n:"Winter '22 Pelzsocken", buy:0, sell:0, markt:0}, {ev:"Winter 2022", n:"Winter '22 Wichtel Säbel", buy:0, sell:0, markt:0}, {ev:"Winter 2022", n:"Winter '22 Wichtel Pickel", buy:0, sell:0, markt:0}, {ev:"Winter 2022", n:"Winter '22 Wichtel Beil", buy:0, sell:0, markt:0}, {ev:"Winter 2022", n:"Winter '22 Wichtel Spaten", buy:0, sell:0, markt:0}, {ev:"Winter 2022", n:"Winter '22 Wichtel Compoundbogen", buy:0, sell:0, markt:0}, {ev:"Winter 2022", n:"Adventsbox", buy:0, sell:0, markt:0}, {ev:"Winter 2022", n:"Winter '22 Schneeschaufel", buy:0, sell:0, markt:0}, {ev:"Winter 2022", n:"Snowflake", buy:0, sell:0, markt:0}, {ev:"Winter 2022", n:"Winter '22 Elsas Elytra", buy:0, sell:0, markt:0}, {ev:"Winter 2022", n:"Winter '22 Fimbulwinter", buy:0, sell:0, markt:0}, {ev:"Winter 2022", n:"Winter '22 Samthandschuhe", buy:0, sell:0, markt:0}, {ev:"Winter 2022", n:"Winter '22 Ghasthunter", buy:0, sell:0, markt:0}, {ev:"Winter 2022", n:"Winter '22 Antiker Brustpanzer", buy:0, sell:0, markt:0}, {ev:"Winter 2022", n:"Winter '22 Wardencalibur", buy:0, sell:0, markt:0}, {ev:"Winter 2022", n:"Winter '22 Festliche Sense", buy:0, sell:0, markt:0}, {ev:"Winter 2022", n:"Winter '22 Schneeballbrücke", buy:0, sell:0, markt:0}, {ev:"Winter 2022", n:"Winter '22 Tannenbaumschreck", buy:0, sell:0, markt:0}, {ev:"Winter 2022", n:"Winter '22 Adventskranz", buy:0, sell:0, markt:0},
        {ev:"Winter 2023", n:"Winter '23 Schnee Zweihänder", buy:0, sell:0, markt:0}, {ev:"Winter 2023", n:"Winter '23 Schnee Krampen", buy:0, sell:0, markt:0}, {ev:"Winter 2023", n:"Winter '23 Schnee Barte", buy:0, sell:0, markt:0}, {ev:"Winter 2023", n:"Winter '23 Schnee Schippe", buy:0, sell:0, markt:0}, {ev:"Winter 2023", n:"Winter '23 Schnee Breithacke", buy:0, sell:0, markt:0}, {ev:"Winter 2023", n:"Winter '23 Schnee Deckungsgeber", buy:0, sell:0, markt:0}, {ev:"Winter 2023", n:"Winter '23 Schneeball Kanone", buy:0, sell:0, markt:0}, {ev:"Winter 2023", n:"Winter '23 Adventskerze", buy:0, sell:0, markt:0}, {ev:"Winter 2023", n:"Winter '23 Weihnachts-Hut", buy:0, sell:0, markt:0}, {ev:"Winter 2023", n:"Weihnachtsschachtel", buy:0, sell:0, markt:0}, {ev:"Winter 2023", n:"Winter '23 Winter Knaller", buy:0, sell:0, markt:0}, {ev:"Winter 2023", n:"Hecamus' Futter", buy:0, sell:0, markt:0}, {ev:"Winter 2023", n:"Björn", buy:0, sell:0, markt:0}, {ev:"Winter 2023", n:"Winter '23 Vereiste Elytra", buy:0, sell:0, markt:0}, {ev:"Winter 2023", n:"Winter '23 Wintermantel", buy:0, sell:0, markt:0}, {ev:"Winter 2023", n:"Winter '23 Inuit Schuhe", buy:0, sell:0, markt:0}, {ev:"Winter 2023", n:"Winter '23 Piglin Diplomatiehelm", buy:0, sell:0, markt:0}, {ev:"Winter 2023", n:"Winter '23 Grinch's Klinge", buy:0, sell:0, markt:0}, {ev:"Winter 2023", n:"Winter '23 Eiskratzer", buy:0, sell:0, markt:0}, {ev:"Winter 2023", n:"Winter '23 Elsas Zauberstab", buy:0, sell:0, markt:0}, {ev:"Winter 2023", n:"Winter '23 Spekulatius Magnet", buy:0, sell:0, markt:0}, {ev:"Winter 2023", n:"Winter '23 Fichten Saatgut", buy:0, sell:0, markt:0}, {ev:"Winter 2023", n:"Weihnachtsdeko", buy:0, sell:0, markt:0},
        {ev:"Winter 2024", n:"Winter '24 Wolfsmaske", buy:0, sell:0, markt:0}, {ev:"Winter 2024", n:"Winter '24 Pelzüberwurf", buy:0, sell:0, markt:0}, {ev:"Winter 2024", n:"Winter '24 Fellhose", buy:0, sell:0, markt:0}, {ev:"Winter 2024", n:"Winter '24 Fellschuhe", buy:0, sell:0, markt:0}, {ev:"Winter 2024", n:"Winter '24 Wolf Dolch", buy:0, sell:0, markt:0}, {ev:"Winter 2024", n:"Winter '24 Wolf Spitzhacke", buy:0, sell:0, markt:0}, {ev:"Winter 2024", n:"Winter '24 Wolf Spaltaxt", buy:0, sell:0, markt:0}, {ev:"Winter 2024", n:"Winter '24 Wolf Schaufel", buy:0, sell:0, markt:0}, {ev:"Winter 2024", n:"Winter '24 Wolf Hacke", buy:0, sell:0, markt:0}, {ev:"Winter 2024", n:"Winter '24 Wolf Holzbogen", buy:0, sell:0, markt:0}, {ev:"Winter 2024", n:"Winter '24 Wolf Dreizack", buy:0, sell:0, markt:0}, {ev:"Winter 2024", n:"Winter '24 Wolf Hammer", buy:0, sell:0, markt:0}, {ev:"Winter 2024", n:"Winter '24 Schnee Kanone", buy:0, sell:0, markt:0}, {ev:"Winter 2024", n:"Winter '24 Obsidian Railgun", buy:0, sell:0, markt:0}, {ev:"Winter 2024", n:"Winter '24 Gravitations Axt", buy:0, sell:0, markt:0}, {ev:"Winter 2024", n:"Winter '24 Skin Auswahltruhe", buy:0, sell:0, markt:0}, {ev:"Winter 2024", n:"Winter '24 Kiefer Plattenspieler", buy:0, sell:0, markt:0}, {ev:"Winter 2024", n:"Winter '24 Santa's Schlitten", buy:0, sell:0, markt:0}, {ev:"Winter 2024", n:"Sternkiste", buy:0, sell:0, markt:0}, {ev:"Winter 2024", n:"Leckerli Dose", buy:0, sell:0, markt:0}, {ev:"Winter 2024", n:"Winter '24 Frost Stern Knaller", buy:0, sell:0, markt:0}, {ev:"Winter 2024", n:"Winter '24 Krampus Kohle", buy:0, sell:0, markt:0}, {ev:"Winter 2024", n:"Winter '24 Lebkuchenhaus", buy:0, sell:0, markt:0}, {ev:"Winter 2024", n:"Winter '24 Weihnachtsgeschichten", buy:0, sell:0, markt:0}, {ev:"Winter 2024", n:"Winter '24 Weihnachtskiste", buy:0, sell:0, markt:0},
        {ev:"Winter 2025", n:"Winter '25 Nussknacker Kopf", buy:0, sell:0, markt:0}, {ev:"Winter 2025", n:"Winter '25 Nussknacker Torso", buy:0, sell:0, markt:0}, {ev:"Winter 2025", n:"Winter '25 Nussknacker Beine", buy:0, sell:0, markt:0}, {ev:"Winter 2025", n:"Winter '25 Nussknacker Füße", buy:0, sell:0, markt:0}, {ev:"Winter 2025", n:"Winter '25 Knackerrapier", buy:0, sell:0, markt:0}, {ev:"Winter 2025", n:"Winter '25 Knackerspitzhacke", buy:0, sell:0, markt:0}, {ev:"Winter 2025", n:"Winter '25 Knackeraxt", buy:0, sell:0, markt:0}, {ev:"Winter 2025", n:"Winter '25 Knackerfeldhacke", buy:0, sell:0, markt:0}, {ev:"Winter 2025", n:"Winter '25 Knackerangelrute", buy:0, sell:0, markt:0}, {ev:"Winter 2025", n:"Winter '25 Knackerbogen", buy:0, sell:0, markt:0}, {ev:"Winter 2025", n:"Winter '25 Schneeschaufel", buy:0, sell:0, markt:0}, {ev:"Winter 2025", n:"Winter '25 Skin Auswahltruhe", buy:0, sell:0, markt:0}, {ev:"Winter 2025", n:"Frosti's Geheimversteck", buy:0, sell:0, markt:0}, {ev:"Winter 2025", n:"Kakaobox", buy:0, sell:0, markt:0}, {ev:"Winter 2025", n:"Winter '25 Spekulatius Kanone", buy:0, sell:0, markt:0}, {ev:"Winter 2025", n:"DirtyBirdy's Vogelfutter", buy:0, sell:0, markt:0}, {ev:"Winter 2025", n:"Frosti", buy:0, sell:0, markt:0}, {ev:"Winter 2025", n:"Super Spekulatius", buy:0, sell:0, markt:0}, {ev:"Winter 2025", n:"Winter '25 Honighahn", buy:0, sell:0, markt:0}
    ];

    let items = [];

    function login() {
        if(document.getElementById('pw').value === ADMIN_PW) {
            isAdmin = true;
            document.getElementById('loginArea').style.display = 'none';
            document.getElementById('adminArea').style.display = 'block';
            document.getElementById('adminControls').style.display = 'block';
            document.getElementById('adminTh').style.display = 'table-cell';
            render();
        } else alert("Passwort falsch!");
    }

    function render() {
        const q = document.getElementById('searchInput').value.toLowerCase();
        const filtered = items.filter(i => i.n.toLowerCase().includes(q) || i.ev.toLowerCase().includes(q));
        
        filtered.sort((a,b) => {
            let v1 = a[sCol], v2 = b[sCol];
            if(typeof v1 === 'string') { v1 = v1.toLowerCase(); v2 = v2.toLowerCase(); }
            if (v1 < v2) return sAsc ? -1 : 1;
            if (v1 > v2) return sAsc ? 1 : -1;
            return 0;
        });

        const body = document.getElementById('tableBody');
        body.innerHTML = filtered.map(i => {
            const realIdx = items.indexOf(i);
            const profit = (i.sell || 0) - (i.buy || 0);
            return `<tr>
                <td style="color:var(--muted)">${i.ev}</td>
                <td><strong>${i.n}</strong></td>
                <td style="color:var(--danger)">${(i.buy||0).toLocaleString()} $</td>
                <td style="color:var(--primary)">${(i.sell||0).toLocaleString()} $</td>
                <td style="color:white; font-weight:bold;">${(i.markt||0).toLocaleString()} $</td>
                <td style="color:${profit >= 0 ? 'var(--success)' : 'var(--danger)'}; font-weight:bold;">${profit.toLocaleString()} $</td>
                ${isAdmin ? `<td>
                    <button class="btn btn-p" style="padding:4px 8px; font-size:0.7rem;" onclick="openEdit(${realIdx})">EDIT</button>
                    <button class="btn btn-d" style="padding:4px 8px; font-size:0.7rem;" onclick="deleteItem(${realIdx})">X</button>
                </td>` : ''}
            </tr>`;
        }).join('');
        document.getElementById('counter').innerText = filtered.length + " Items";
    }

    function resort(c) { sAsc = (sCol === c) ? !sAsc : true; sCol = c; render(); }

    // Modal Logic
    function openEdit(i) {
        editIdx = i;
        document.getElementById('mTitle').innerText = items[i].n;
        document.getElementById('mBuy').value = items[i].buy;
        document.getElementById('mSell').value = items[i].sell;
        document.getElementById('mMarkt').value = items[i].markt;
        document.getElementById('editModal').style.display = 'flex';
    }

    function openAddModal() { document.getElementById('addModal').style.display = 'flex'; }
    function closeModal(id) { document.getElementById(id).style.display = 'none'; }
    function clearValues() { document.getElementById('mBuy').value = 0; document.getElementById('mSell').value = 0; document.getElementById('mMarkt').value = 0; }

    function save() {
        items[editIdx].buy = parseInt(document.getElementById('mBuy').value) || 0;
        items[editIdx].sell = parseInt(document.getElementById('mSell').value) || 0;
        items[editIdx].markt = parseInt(document.getElementById('mMarkt').value) || 0;
        dbSave(); render(); closeModal('editModal');
    }

    function addNewItem() {
        const ev = document.getElementById('addEv').value;
        const n = document.getElementById('addN').value;
        if(ev && n) {
            items.push({ev, n, buy:0, sell:0, markt:0});
            dbSave(); render(); closeModal('addModal');
            document.getElementById('addEv').value = ""; document.getElementById('addN').value = "";
        } else alert("Bitte Event und Name ausfüllen!");
    }

    function deleteItem(i) {
        if(confirm(`"${items[i].n}" wirklich löschen?`)) {
            items.splice(i, 1);
            dbSave(); render();
        }
    }

    function dbSave() { localStorage.setItem('mixel_db_final_v10', JSON.stringify(items)); }
    function resetToCode() { if(confirm("Lokale Änderungen löschen und Code-Daten laden?")) { localStorage.removeItem('mixel_db_final_v10'); location.reload(); } }

    function exportFullHTML() {
        let html = document.documentElement.outerHTML;
        const marker = "const rawData = [";
        const parts = html.split(marker);
        const endPart = parts[1].split("];");
        const newHTML = parts[0] + marker + "\n        " + JSON.stringify(items, null, 4).slice(1,-1) + "\n    ];" + endPart[1];
        
        const blob = new Blob([newHTML], {type: 'text/html'});
        const a = document.createElement('a');
        a.href = URL.createObjectURL(blob);
        a.download = "mixel_preise_update.html";
        a.click();
    }

    // Init
    const stored = localStorage.getItem('mixel_db_final_v10');
    items = (stored) ? JSON.parse(stored) : [...rawData];
    render();
</script>
</body>
</html>
