# Agregar los datos

### Datos de contratos

Para esta parte, deberás tener el listado de contratos que quieres visualizar en el formato del estándar de contrataciones abiertas [OCDS](http://standard.open-contracting.org/latest/en/). Puedes obtener el listado de alguna de las fuentes OCDS utilizando la herramienta [Kingfisher](https://github.com/open-contracting/kingfisher) para guardarlo en disco. Para más información puedes revisar la [documentación completa](https://ocdskingfisher.readthedocs.io/en/latest/) de Kingfisher.

El estándar OCDS es un modelo universal para la publicación y análisis de datos de procesos de contratación. Los datos publicados bajo el estándar se encuentran en el formato JSON y pueden presentarse en dos formatos: [release](http://standard.open-contracting.org/latest/en/schema/reference/) y [record](http://standard.open-contracting.org/latest/en/schema/records_reference/). Generalmente, los datos OCDS se publican en listados de procesos de contratación conocidos como packages. Es posible encontrar [paquetes de releases](http://standard.open-contracting.org/latest/en/schema/release_package/) y [paquetes de records](http://standard.open-contracting.org/latest/en/schema/record_package/) publicados por los gobiernos de ciertos países.

El archivo de datos debe contener un listado de records o releases de los procesos de contratación. Para lograr que el archivo sea sólo el listado de records o releases dentro de un array, puede ser necesario manipular el contenido del archivo. Para esto recomendamos la herramienta ocdskit y en particular jq. El archivo final de datos debe contener una de las siguientes estructuras:

- **Release package:** un objeto "releases" que contiene un listado en forma de array (entre corchetes []) cuyos elementos son releases individuales.
    ```
    {
    	"releases": [
            { (release 1) },
            { (release 2) },
            ...
            { (release n) }
        ]
    }
    ```

- **Record package:** un objeto "records" que contiene un array cuyos elementos son objetos de tipo record. Cada record a su vez contiene un objeto "releases" con el formato de un release package.
    ```
    {
    	"records": [
            {
                "releases": [
                    { (release 1) },
                    { (release 2) },
                    ...
                    { (release n) }
                ]
            },
            { (record 2) },
            ...
            { (record n) }
        ]
    }
    ```

Nota: dentro de cada release, es necesario que ciertos campos contengan algún valor para que los gráficos se desplieguen de manera correcta. Los campos obligatorios son:
- *ocid*
- *tender.title*
- *tender.mainProcurementCategory*
- *tender.procurementMethodDetails*
- *contracts.value.amount*
- *contracts.value.currency*
- Dentro del campo parties, al menos uno con *role: ["supplier"]* y con valores para los campos de *id* y *name*

Los datos deberán ser colocados en un archivo con el nombre **contracts.json** y el archivo ubicado en la ruta *assets/data/*. Para subir el archivo al repositorio de Github, debes ubicarte en la página principal del repositorio en tu navegador. En el listado de archivos, primero haz click en la carpeta *assets* y en la pantalla siguiente haz click en la carpeta *data*. Una vez estés adentro de la carpeta *data* debes hacer click en el botón "Upload files", ubicado sobre la tabla del listado de archivos al lado derecho de la pantalla. Esto te llevará a una pantalla desde la cual puedes elegir el archivo desde tu computadora o arrastrarlo hacia la ventana del navegador. Si no ves el botón para subir archivos debes iniciar sesión en Github con tu usuario y contraseña.

### Datos de beneficiarios reales

Para complementar los datos de procesos de contratación es posible indicar la manera como éstos se relacionan con las personas y empresas que se encuentran detrás de las entidades que aparecen en los datos publicados bajo OCDS. Las relaciones se expresan como un árbol de jerarquías, en el cual se establecen relaciones entre empresas (una empresa matriz y sus subsidiarias) o entre empresas y personas (accionistas y miembros de juntas directivas de una empresa). Este árbol de jerarquías permite establecer quiénes son los beneficiarios reales de los procesos de contratación analizados.

> Un Beneficiario Real es la persona física o natural quien, directa o indirectamente y en última instancia, posee, influencia, controla y/o se beneficia de al menos el 5% de un activo mediante un vehículo corporativo, sociedad mercantil o fideicomiso. Más información sobre Beneficiarios Reales, [aquí](https://www.colaboratorio.org/beneficiarios-reales-en-mexico/).

Para crear el conjunto de datos de beneficiarios reales debes usar la plantilla disponible en *assets/data/BO-template.csv*, editarla en un software de planilla de cálculo (LibreOffice Calc, MS Excel, Google Spreadsheets u otro) y modificar los valores. El archivo contiene las siguientes columnas:

1. **NOMBRE:** el nombre de la entidad (persona o empresa) tal y como aparece en los datos OCDS.
2. **TIPO_ENTIDAD:** las palabras "*empresa*" o "*persona*" según la entidad de la primera columna.
3. **RELACIONADO_CON:** la entidad con la cual se desea establecer la relación.
4. **TIPO_RELACION:** cómo se relaciona la entidad de la primera columna con la entidad de la tercera columna. Puede contener los siguientes valores:
    - "*parent*" para establecer que la empresa en la tercera columna es subsidiaria de la empresa en la primera columna.
    - "*shareholder*" cuando la persona de la primera columna es accionista de la empresa en la tercera columna.
    - "*boardmember*" si la persona en la primera columna está en la junta directiva de la empresa en la tercera columna.
5. **PUESTO:** el nombre del puesto que ocupa la persona. Sólo aplica para un TIPO_RELACION con un valor de "*boardmember*".

Cada fila del archivo representa una rama del árbol de jerarquías, partiendo de una fila raíz que contendrá solamente el nombre de la empresa en la primera columna y la palabra "empresa" en la segunda. Es importante que en cada fila pongas los nombres de las empresas tal cual están en el listado de contratos, para que se pueda vincular la información en el gráfico. Como ejemplo, puedes tomar la siguiente jerarquía:

- **Empresa A** (1) aparece en un proceso de contratación.
- **Empresa B** (2) es la empresa matriz de **Empresa A**.
- **Persona C** (3) y **Persona D** (4) son accionistas de **Empresa B**.
- **Persona E** (5) y **Persona F** (6) son el **Presidente** y el **Vicepresidente** de la junta directiva de **Empresa B**.

Deberás llenar el archivo de la siguiente manera, con la información de cada entidad en una fila separada:

![Ejemplo CSV](csvtable.png "Ejemplo CSV")

Repite el mismo proceso para cada jerarquía de empresas de la que tengas datos. Una vez tu archivo esté completo, puedes utilizar el mismo procedimiento de la sección anterior para subirlo a la carpeta *assets/data/*. Para este archivo deberás utilizar el nombre **owners.csv**.
