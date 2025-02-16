<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Free Fire Visit Adder</title>
</head>
<body>
    <h2>Enter UID to Add Visits</h2>
    <input type="text" id="uidInput" placeholder="Enter UID">
    <button onclick="addVisits()">Submit</button>
    <p id="response"></p>

    <script>
        async function addVisits() {
            const uid = document.getElementById('uidInput').value;
            if (!uid) {
                document.getElementById('response').innerText = "Please enter a UID.";
                return;
            }

            const url = `https://freefire.scaninfo.net/visit/?uid=${uid}`;
            try {
                const response = await fetch(url);
                const data = await response.json();
                document.getElementById('response').innerText = `Response: ${data.message}`;
            } catch (error) {
                document.getElementById('response').innerText = "Error fetching data.";
            }
        }
    </script>
</body>
</html>
