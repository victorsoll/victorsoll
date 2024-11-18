<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Application Achat-Revente Dashboard</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #1a1a1a;
            color: #fff;
            margin: 0;
            padding: 0;
        }

        .container {
            padding: 16px;
        }

        .dashboard {
            display: flex;
            flex-wrap: wrap;
            gap: 16px;
            margin-bottom: 16px;
        }

        .card {
            background-color: #333;
            padding: 16px;
            border-radius: 8px;
            flex: 1;
            min-width: 150px;
            text-align: center;
        }

        .card h3 {
            margin: 0 0 8px;
            font-size: 1.2em;
        }

        .card p {
            margin: 0;
            font-size: 1.5em;
            font-weight: bold;
        }

        .selectors {
            display: flex;
            gap: 16px;
            margin-bottom: 16px;
        }

        select {
            padding: 8px 16px;
            border-radius: 20px;
            background-color: #444;
            color: #fff;
            border: none;
            font-size: 1em;
            cursor: pointer;
        }

        table {
            width: 100%;
            border-collapse: collapse;
            margin-top: 16px;
        }

        table th, table td {
            border: 1px solid #444;
            padding: 8px;
            text-align: center;
        }

        table th {
            background-color: #333;
            color: #fff;
        }

        table td {
            background-color: #222;
        }

        .form-container {
            background-color: #333;
            padding: 16px;
            border-radius: 8px;
            margin-bottom: 16px;
        }

        .form-container input, .form-container button {
            width: 100%;
            padding: 8px;
            margin: 8px 0;
            border-radius: 4px;
            border: none;
            outline: none;
        }

        .form-container button {
            background-color: red;
            color: #fff;
            cursor: pointer;
        }

        .delete-button {
            background-color: red;
            color: white;
            border: none;
            padding: 6px 12px;
            border-radius: 4px;
            cursor: pointer;
        }

        .delete-button:hover {
            background-color: darkred;
        }

        .edit-button {
            background-color: #007bff;
            color: white;
            border: none;
            padding: 6px 12px;
            border-radius: 4px;
            cursor: pointer;
        }

        .edit-button:hover {
            background-color: #0056b3;
        }

        .action-buttons {
            display: flex;
            flex-direction: column; /* Les boutons s'empilent verticalement */
            gap: 8px; /* Espace entre les boutons */
        }
    </style>
    <script>
        let articles = JSON.parse(localStorage.getItem('articles')) || []; // Récupérer les articles depuis localStorage au chargement de la page

        function ajouterArticle() {
            const nom = document.getElementById('nom').value;
            const marque = document.getElementById('marque').value;
            const prixAchat = parseFloat(document.getElementById('prixAchat').value);
            const prixVente = document.getElementById('prixVente').value ? parseFloat(document.getElementById('prixVente').value) : "Stock";
            const date = document.getElementById('date').value;

            if (nom && marque && date && !isNaN(prixAchat)) {
                const article = {
                    nom,
                    marque,
                    prixAchat,
                    prixVente,
                    date
                };
                articles.push(article);
                localStorage.setItem('articles', JSON.stringify(articles)); // Sauvegarder les articles dans localStorage
                document.getElementById('articleForm').reset();
                afficherArticles(); // Met à jour le tableau
            } else {
                alert('Veuillez remplir tous les champs correctement.');
            }
        }

        function modifierArticle(index) {
            const prixVente = prompt("Entrez le prix de vente pour cet article :");
            if (prixVente !== null && !isNaN(prixVente)) {
                articles[index].prixVente = parseFloat(prixVente); // Met à jour le prix de vente
                localStorage.setItem('articles', JSON.stringify(articles)); // Sauvegarde dans localStorage
                afficherArticles(); // Rafraîchit le tableau
            } else if (prixVente !== null) {
                alert("Veuillez entrer un prix de vente valide.");
            }
        }

        function supprimerArticle(index) {
            if (confirm("Êtes-vous sûr de vouloir supprimer cet article ?")) {
                articles.splice(index, 1); // Supprime l'article de la liste
                localStorage.setItem('articles', JSON.stringify(articles)); // Met à jour le localStorage
                afficherArticles(); // Rafraîchit le tableau
            }
        }

        function filtrerArticlesParDate() {
            const mois = document.getElementById('mois').value;
            const annee = document.getElementById('annee').value;

            return articles.filter(article => {
                const articleDate = new Date(article.date);
                const articleMois = articleDate.getMonth() + 1;
                const articleAnnee = articleDate.getFullYear();

                return (
                    (mois === "Tous" || articleMois === parseInt(mois)) &&
                    (annee === "Tous" || articleAnnee === parseInt(annee))
                );
            });
        }

        function calculerStatistiques(articlesFiltres) {
            let totalAchat = 0;
            let totalVente = 0;

            articlesFiltres.forEach(article => {
                totalAchat += article.prixAchat;
                if (article.prixVente !== "Stock") {
                    totalVente += parseFloat(article.prixVente);
                }
            });

            const benefice = totalVente - totalAchat;
            const margeMoyenne = totalAchat > 0 ? (totalVente / totalAchat).toFixed(2) : "0";

            // Mise à jour des valeurs affichées
            document.getElementById('totalAchats').textContent = totalAchat.toFixed(2) + " €";
            document.getElementById('totalVentes').textContent = totalVente.toFixed(2) + " €";
            document.getElementById('benefice').textContent = benefice.toFixed(2) + " €";
            document.getElementById('margeMoyenne').textContent = margeMoyenne + "x";
        }

        function afficherArticles() {
            const tableau = document.getElementById('listeArticles');
            tableau.innerHTML = ''; // Effacer le contenu précédent

            const headerRow = `
                <tr>
                    <th>Produit</th>
                    <th>Marque</th>
                    <th>Prix d'achat</th>
                    <th>Prix de vente</th>
                    <th>Bénéfice</th>
                    <th>Action</th>
                </tr>
            `;
            tableau.innerHTML = headerRow;

            const articlesFiltres = filtrerArticlesParDate();

            // Boucle sur chaque article
            articlesFiltres.forEach((article, index) => {
                const benefice = article.prixVente !== "Stock" ? (article.prixVente - article.prixAchat).toFixed(2) : "-";
                const row = document.createElement('tr');

                row.innerHTML = `
                    <td>${article.nom}</td>
                    <td>${article.marque}</td>
                    <td style="color: red;">${article.prixAchat.toFixed(2)} €</td>
                    <td style="color: ${article.prixVente === "Stock" ? "gray" : "red"};">
                        ${article.prixVente === "Stock" ? "Stock" : article.prixVente.toFixed(2) + " €"}
                    </td>
                    <td style="color: ${benefice > 0 ? "green" : benefice < 0 ? "red" : "gray"};">
                        ${benefice === "-" ? "-" : benefice + " €"}
                    </td>
                    <td class="action-buttons">
                        <button class="edit-button" onclick="modifierArticle(${index})">Modifier</button>
                        <button class="delete-button" onclick="supprimerArticle(${index})">Supprimer</button>
                    </td>
                `;

                tableau.appendChild(row); // Ajouter la ligne au tableau
            });

            calculerStatistiques(articlesFiltres); // Recalculer les statistiques
        }

        // Appeler afficherArticles() au chargement de la page pour afficher les articles sauvegardés
        window.onload = afficherArticles;
    </script>
</head>
<body>
    <div class="container">
        <h1>DASHBOARD</h1>

        <!-- Sélecteur d'année et mois -->
        <div class="selectors">
            <select id="annee" onchange="afficherArticles()">
                <option value="Tous">Tous</option>
                <option value="2024">2024</option>
                <option value="2025">2025</option>
                <option value="2026">2026</option>
            </select>

            <select id="mois" onchange="afficherArticles()">
                <option value="Tous">Tous</option>
                <option value="01">Janvier</option>
                <option value="02">Février</option>
                <option value="03">Mars</option>
                <option value="04">Avril</option>
                <option value="05">Mai</option>
                <option value="06">Juin</option>
                <option value="07">Juillet</option>
                <option value="08">Août</option>
                <option value="09">Septembre</option>
                <option value="10">Octobre</option>
                <option value="11">Novembre</option>
                <option value="12">Décembre</option>
            </select>
        </div>

        <div class="dashboard">
            <div class="card">
                <h3>Bénéfice</h3>
                <p id="benefice">0 €</p>
            </div>
            <div class="card">
                <h3>Total achats</h3>
                <p id="totalAchats">0 €</p>
            </div>
            <div class="card">
                <h3>Total ventes (CA)</h3>
                <p id="totalVentes">0 €</p>
            </div>
            <div class="card">
                <h3>Marge moyenne</h3>
                <p id="margeMoyenne">0x</p>
            </div>
        </div>

        <div class="form-container">
            <h2>Ajouter un article</h2>
            <form id="articleForm">
                <input type="text" id="nom" placeholder="Nom de l'article" required>
                <input type="text" id="marque" placeholder="Marque" required>
                <input type="number" step="0.01" id="prixAchat" placeholder="Prix d'achat (€)" required>
                <input type="number" step="0.01" id="prixVente" placeholder="Prix de vente (€)">
                <input type="date" id="date" required>
                <button type="button" onclick="ajouterArticle()">Ajouter</button>
            </form>
        </div>

        <h2>Liste des articles:</h2>
        <table id="listeArticles"></table>
    </div>
</body>
</html>
