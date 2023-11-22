import uvicorn
import requests

from fastapi import FastAPI
from fastapi.responses import HTMLResponse  # Necesario para enviar html
from fastapi import APIRouter
from fastapi.security import HTTPBasic

app = FastAPI()
router = APIRouter()
security = HTTPBasic()


@app.get("/", response_class=HTMLResponse)
async def get_html():
    server_host = "172.16.143.162"
    html_content = f"""
    <!DOCTYPE html>
    <html lang="es">

    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>Formulario Simple</title>
        <style>
            body {{
                font-family: Arial, sans-serif;
                background-color: #f4f4f4;
                margin: 0;
                padding: 0;
                display: flex;
                flex-direction: column;
                align-items: center;
                justify-content: center;
                height: 100vh;
            }}

            form {{
                background-color: #fff;
                padding: 20px;
                border-radius: 8px;
                box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
            }}

            label {{
                display: block;
                margin-bottom: 8px;
            }}

            input {{
                width: 100%;
                padding: 8px;
                margin-bottom: 16px;
                box-sizing: border-box;
            }}

            button {{
                background-color: #4caf50;
                color: #fff;
                padding: 10px 15px;
                border: none;
                border-radius: 4px;
                cursor: pointer;
            }}

            button:hover {{
                background-color: #45a049;
            }}
        </style>
    </head>

    <body>
        <p id="explicacion">
            <p>La api que uso es una api de alojamientos turísticos en Mallorca.</p>
            <a href="https://datos.gob.es/es/catalogo/a04003003-alojamientos-turisticos-mallorca">Enlace a la api.</a>
            <p>
                Este html muestra un formulario con las distintas opciones que el usuario puede visualizar.
            </p>
            <p>
                Filtrar: Muestra la tabla filtrada por el valor que introduzcas en el input.
            </p>
            <p>
                Mostrar todo: Muestra toda la tabla.
            </p>
            <p>
                Nº de alojamientos/municipio: Muestra una tabla con la cantidad de alojamientos de cada municipio.
            </p>
        </p>

        <form>
            <label for="texto">Filtro por municipio:</label>
            <input type="text" id="municipio" name="texto" placeholder="palma">
            <br>
            <button type="submit" onclick="filtrar()">Filtrar</button>
            <button type="button" onclick="window.location.href='http://{server_host}:800/tabla'">Mostrar todo</button>
            <button type="button" onclick="window.location.href='http://{server_host}:800/Total_alojamientos'">Nº de alojamientos/municipio</button>
        </form>

        <script>
            function filtrar() {{
                municipio = document.getElementById('municipio').value;
                if (municipio == "") {{
                    municipio = "palma";
                }}
                var url = "http://{server_host}:800/get/" + encodeURI(municipio);

                // Redirige a la página con la URL construida
                window.location.href = url;

                // Evita que el formulario se envíe (puedes eliminar esta línea si quieres enviar el formulario)
                event.preventDefault();
            }}
        </script>
    </body>

    </html>
    """
    return HTMLResponse(content=html_content)


@app.get("/tabla", response_class=HTMLResponse)
async def get_table():
    api_url = "https://catalegdades.caib.cat/resource/j2yj-e83g.json"
    response = requests.get(api_url, timeout=10)
    if response.status_code == 200:
        data = response.json()
        html_content = """
        <html>
            <head>
                <title>Allotjaments turístics Mallorca</title>
            </head>
            <body>
                <table border="1">
                    <tr>
                        <th>Signatura</th>
                        <th>Denominació comercial</th>
                        <th>Grup</th>
                        <th>Subgrup</th>
                        <th>Inici d'activitat</th>
                        <th>Estat</th>
                        <th>Municipi</th>
                        <th>Localitat</th>
                        <th>Direcció</th>
                        <th>UTM_X</th>
                        <th>UTM_Y</th>
                        <th>Places</th>
                        <th>Unitats</th>
                        <th>Explotador</th>
                        <th>Geocoded column</th>
                        <th>Coordinates</th>
                    </tr>
        """
        for item in data:
            html_content += f"""
                <tr>
                    <td>{item.get('signatura', 'N/A')}</td>
                    <td>{item.get('denominaci_comercial', 'N/A')}</td>
                    <td>{item.get('grup', 'N/A')}</td>
                    <td>{item.get('subgrup', 'N/A')}</td>
                    <td>{item.get('inici_d_activitat', 'N/A')}</td>
                    <td>{item.get('estat', 'N/A')}</td>
                    <td>{item.get('municipi', 'N/A')}</td>
                    <td>{item.get('localitat', 'N/A')}</td>
                    <td>{item.get('direcci', 'N/A')}</td>
                    <td>{item.get('utm_x')}</td>    
                    <td>{item.get('utm_y', 'N/A')}</td>
                    <td>{item.get('places', 'N/A')}</td>
                    <td>{item.get('unitats', 'N/A')}</td>
                    <td>{item.get('explotador_s', 'N/A')}</td>
                    <td>{item.get('geocoded_column', {}).get('type', 'N/A')}</td>
                    <td>{item.get('geocoded_column', {}).get('coordinates', 'N/A')}</td>
                </tr>
            """
        html_content += """
                </table>
            </body>
        </html>
        """
        return HTMLResponse(content=html_content, status_code=200)
    else:
        return HTMLResponse(content="<p>Error al acceder a la API</p>", status_code=500)


# MUESTRO LOS MUNICIPIOS QUE INTRODUCE POR LA RUTA
@app.get("/get/{user_input}", response_class=HTMLResponse)
async def get_municipio(user_input: str):
    api_url = "https://catalegdades.caib.cat/resource/j2yj-e83g.json"
    response = requests.get(api_url, timeout=10)
    if response.status_code == 200:
        data = response.json()
        html_content = """
        <html>
            <head>
                <title>Allotjaments turístics {user_input}</title>
            </head>
            <body>
                <table border="1">
                    <tr>
                        <th>Signatura</th>
                        <th>Denominació comercial</th>
                        <th>Grup</th>
                        <th>Subgrup</th>
                        <th>Inici d'activitat</th>
                        <th>Estat</th>
                        <th>Municipi</th>
                        <th>Localitat</th>
                        <th>Direcció</th>
                        <th>UTM_X</th>
                        <th>UTM_Y</th>
                        <th>Places</th>
                        <th>Unitats</th>
                        <th>Explotador</th>
                        <th>Geocoded column</th>
                        <th>Coordinates</th>
                    </tr>
        """
        for item in data:
            if item.get("municipi") == user_input.upper():
                html_content += f"""
                <tr>
                    <td>{item.get('signatura', 'N/A')}</td>
                    <td>{item.get('denominaci_comercial', 'N/A')}</td>
                    <td>{item.get('grup', 'N/A')}</td>
                    <td>{item.get('subgrup', 'N/A')}</td>
                    <td>{item.get('inici_d_activitat', 'N/A')}</td>
                    <td>{item.get('estat', 'N/A')}</td>
                    <td>{item.get('municipi', 'N/A')}</td>
                    <td>{item.get('localitat', 'N/A')}</td>
                    <td>{item.get('direcci', 'N/A')}</td>
                    <td>{item.get('utm_x')}</td>    
                    <td>{item.get('utm_y', 'N/A')}</td>
                    <td>{item.get('places', 'N/A')}</td>
                    <td>{item.get('unitats', 'N/A')}</td>
                    <td>{item.get('explotador_s', 'N/A')}</td>
                    <td>{item.get('geocoded_column', {}).get('type', 'N/A')}</td>
                    <td>{item.get('geocoded_column', {}).get('coordinates', 'N/A')}</td>
                </tr>
            """
        html_content += """
                </table>
            </body>
        </html>
        """
        return HTMLResponse(content=html_content, status_code=200)
    else:
        return HTMLResponse(content="<p>Error al acceder a la API</p>", status_code=500)


@app.get("/Total_alojamientos", response_class=HTMLResponse)
async def get_total():
    api_url = "https://catalegdades.caib.cat/resource/j2yj-e83g.json"
    response = requests.get(api_url, timeout=10)

    if response.status_code == 200:
        data = response.json()
        municipio_contadores = {}

        # Contar la cantidad de alojamientos por municipio
        for item in data:
            municipio = item.get("municipi", "N/A").upper()
            if municipio in municipio_contadores:
                municipio_contadores[municipio] += 1
            else:
                municipio_contadores[municipio] = 1

        html_content = """
        <html>
            <head>
                <title>Allotjaments turístics Mallorca</title>
            </head>
            <body>
                <table border="1">
                    <tr>
                        <th>Municipio</th>
                        <th>Total</th>
                    </tr>
        """

        # Mostrar la información de alojamientos por municipio
        for municipio, total in municipio_contadores.items():
            html_content += f"""
                <tr>
                    <td>{municipio}</td>
                    <td>{total}</td>
                </tr>
            """

        html_content += """
                </table>
            </body>
        </html>
        """
        return HTMLResponse(content=html_content, status_code=200)
    else:
        print("Error en la respuesta de la API:", response.status_code)
        return HTMLResponse(content="<p>Error al acceder a la API</p>", status_code=500)


if __name__ == "__main__":
    uvicorn.run(app, host="0.0.0.0", port=800, log_level="info")
