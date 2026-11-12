# Microsoft Sentinel: demo de despliegue desde GitHub

Esta guía muestra el ciclo completo de una consulta de búsqueda: crear el archivo en GitHub, desplegarlo automáticamente y comprobar que aparece y se ejecuta en Microsoft Sentinel.

El repositorio está conectado al workspace `law-sc200-repo-demo`. El workflow de GitHub Actions se ejecuta al hacer push a `main` y despliega contenido al workspace.

## Qué se va a demostrar

1. Se crea una consulta KQL como archivo JSON en el repositorio.
2. Un commit en `main` inicia GitHub Actions.
3. El workflow despliega la consulta en Microsoft Sentinel.
4. La consulta aparece en la lista de consultas del repositorio y se puede ejecutar.

La consulta usa `print` para devolver una fila de demostración. No necesita conectores ni eventos reales; esto sirve para mostrar el flujo de administración y despliegue de contenido.

## Antes de presentar

- Confirma que el workspace `law-sc200-repo-demo` está conectado al repositorio `frevar1975/sentinel`.
- En GitHub, abre la pestaña **Actions** y confirma que el workflow **Deploy Content to law-sc200-repo-demo** está habilitado.
- Usa el portal de Microsoft Defender/Sentinel con acceso al workspace conectado.

## Demo paso a paso

### 1. Crear una consulta en el repositorio

En GitHub, abre el repositorio [frevar1975/sentinel](https://github.com/frevar1975/sentinel).

1. Entra en `SentinelContent/Queries`.
2. Selecciona **Add file → Create new file**.
3. Como nombre, escribe `SentinelContent/Queries/SC200-Demo-RepoQuery-3.json`.
4. Pega este contenido:

~~~json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "workspace": {
      "type": "string"
    }
  },
  "resources": [
    {
      "type": "Microsoft.OperationalInsights/workspaces/savedSearches",
      "apiVersion": "2020-03-01-preview",
      "name": "[concat(parameters('workspace'), '/sc200-demo-repo-query-3')]",
      "properties": {
        "Category": "Hunting Queries",
        "DisplayName": "SC-200 Demo 3 - Consulta creada en GitHub",
        "Query": "print Demo = 'Consulta creada en GitHub y desplegada en Sentinel'",
        "Tags": [
          {
            "Name": "Description",
            "Value": "Consulta de demostración para mostrar el despliegue desde GitHub."
          },
          {
            "Name": "Tactics",
            "Value": "Discovery"
          }
        ]
      }
    }
  ]
}
~~~

5. En **Commit changes**, usa un mensaje como `Agregar consulta demo 3` y confirma el cambio directamente en la rama `main`.

El nombre interno `sc200-demo-repo-query-3` debe ser único en el workspace. Para repetir la demostración, aumenta el sufijo en el nombre del archivo, el nombre interno y el título.

### 2. Comprobar el despliegue en GitHub Actions

1. Abre la pestaña **Actions** del repositorio.
2. Abre el run que se inició con el commit recién creado.
3. Espera a que termine con estado **Succeeded** (marca verde).
4. Abre el job **deploy-content** y confirma que el paso **Deploy Content to Microsoft Sentinel** terminó correctamente.

Si no aparece un run nuevo, comprueba que el commit quedó en `main` y que el archivo está dentro de `SentinelContent/Queries`. El workflow está configurado para ejecutarse con cambios del repositorio en `main`.

### 3. Ver la consulta en Microsoft Sentinel

1. Abre Microsoft Sentinel para el workspace `law-sc200-repo-demo`.
2. En el menú, ve a **Threat management → Hunting** y abre la pestaña **Queries** (en español puede mostrarse como **Consultas**).
3. Busca **SC-200 Demo 3 - Consulta creada en GitHub**.
4. Selecciónala y comprueba que el origen del contenido indica **Repositories/Repositorios**.

Esto confirma que Sentinel recibió la consulta desde la conexión del repositorio. No busques el elemento en **Hunts/Búsquedas**: una Hunt es una investigación distinta de una consulta guardada.

### 4. Ejecutar la KQL y mostrar el resultado

Para mostrar una salida visible en pantalla:

1. Abre **Advanced hunting / Búsqueda avanzada** en el mismo workspace.
2. Pega esta consulta en el editor:

~~~kusto
print Demo = 'Consulta creada en GitHub y desplegada en Sentinel'
~~~

3. Selecciona **Run query / Ejecutar consulta**.
4. Muestra el resultado: una fila con la columna `Demo` y el texto de demostración.

La lista de **Queries/Consultas** confirma la presencia del contenido desplegado; el resultado de **Advanced hunting** demuestra la ejecución de la expresión KQL. La consulta `print` no busca actividad ni genera una alerta.

## Guion oral breve

> “La consulta está definida como código en GitHub. Al confirmar el archivo en la rama principal, GitHub Actions ejecuta el workflow de despliegue y publica el contenido en el workspace de Microsoft Sentinel. Aquí verificamos que la consulta aparece con origen Repositories y ejecutamos su KQL para ver el resultado.”

## Si algo no aparece

- **El workflow falla:** abre el paso fallido en Actions y revisa el mensaje del job; confirma que el workspace sigue conectado y que la identidad de despliegue conserva permisos.
- **El workflow termina, pero no ves la consulta:** confirma que estás en el workspace `law-sc200-repo-demo`, actualiza la página de Queries y busca el título completo.
- **Advanced hunting muestra un editor vacío:** pega el KQL de arriba y ejecútalo. La consulta guardada y el editor de Advanced hunting son pantallas distintas.
- **No hay eventos en el workspace:** no afecta a esta prueba, porque `print` produce una fila sin consultar tablas de eventos.
