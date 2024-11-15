<div style="display: flex; width: 100%;">
    <div style="flex: 1; padding: 0px;">
        <p>© Albert Palacios Jiménez, 2024</p>
    </div>
    <div style="flex: 1; padding: 0px; text-align: right;">
        <img src="./assets/ieti.png" height="32" alt="Logo de IETI" style="max-height: 32px;">
    </div>
</div>
<br/>

# Crides a servidors

La comunicació amb servidors Web, normalment es fa a través de crides **Get** i **Post**

## Get

Les crides **Get** demanen dades a un servidor, i passen la informació a través de la *URL* 

- Pot ser que la informació simplement estigui a la pròpia URL sense cap format estàndard:
```text
http://example.com/resource/123
```

- Habitualment es passa la informació després del símbol **?**:
```text
http://example.com/api?param1=value1&param2=value2
```
En aquest exemple la informació que es passa, són les claus i valors:

- clau *param1* amb valor *value1*
- clau *param2* amb valor *value2*

**Característiques**

Les crides **Get** tenen un límit màxim d'informació que correspòn a la llargada màxima de la URL.

Les crides **Get** són visibles per:

- Els altres usuaris de la mateixa xarxa
- La companyia telefònica
- Els governs

Així que **NO S'HA DE POSAR INFORMACIÓ PRIVADA** en una crida **Get**

Normalment les crides web no demanen dades personalitzades que poden canviar l'estat del servidor.

**NodeJS**

En els següent exemples:

- **req** és la petició rebuda del client al servidor, que inclou la informació de la crida *get*

- **res** és l'objecte de resposta a la crida rebuda. On s'inclou l'objecte *.json*

Exemples de crida *Get*:

```javascript
// http://localhost:3000/api?param1=value1&param2=value2
app.get('/api', (req, res) => {
    // Obtenir el valor de "param1"
    const param1 = req.query.param1 

    // Obtenir el valor de "param2"
    const param2 = req.query.param2

    res.json({
        message: 'Dades rebudes',
        param1: param1,
        param2: param2
    })
})
```

Exemple de paràmetres *Get* dinàmics:

```javascript
app.get('/users/:userId/posts/:postId', (req, res) => {
    // Obtenir l'ID de l'usuari
    const userId = req.params.userId 

    // Obtenir l'ID de la publicació
    const postId = req.params.postId 

    res.json({
        message: 'Informació del post',
        userId: userId,
        postId: postId
    })
})
```

## Post

Les crides **Post** demanen dades a un servidor, amb la característica que passen la informació amb el cos de la petició.

La informació es manté oculta si s'utilitza el protocol **https**.

**Característiques**

La quantitat màxima d'informació de les crides **Post** depèn de la configuració del servidor.

S'ha de fer servir crides **Post** al passar informació sensible com contrasenyes o dades dels usuaris. El què fa que el servidor respongui amb estats específics per cada usuari.

Els usus habituals inclouen l'enviament de fitxers, informació personal, interactuar amb recursos o bases de dades ...

**NodeJS**

Exemple de crida *Post*:
```javascript
/*
curl -X POST http://localhost:3000/api \
-H "Content-Type: application/json" \
-d '{"param1": "value1", "param2": "value2"}'
*/
app.post('/api', (req, res) => {
    // Obtenir valors del cos de la petició
    const { param1, param2 } = req.body

    res.json({
        message: 'Dades rebudes',
        param1: param1,
        param2: param2
    })
})
```

## Post amb arxius (multer)

Per gestionar dades *Post* amb arxius es fa servir **multipart/form-data**. Aquest format permet enviar text i fitxers dins d'una mateixa petició.

*NodeJS* ofereix la llibreria **Multer** per treballar amb aquest format. Pots instal·lar-la així:

Per instal·lar-la:
```bash
npm install multer
```

Exemple:

```javascript
/*
curl -X POST http://localhost:3000/upload \
-H "Content-Type: multipart/form-data" \
-F "json={\"key1\":\"value1\",\"key2\":\"value2\"}" \
-F "files=@/path/to/your/file1.txt" \
-F "files=@/path/to/your/file2.txt"
*/
const express = require('express')
const multer = require('multer')
const app = express()

// Crear la carpeta 'uploads' si no existeix
const fs = require('fs')
if (!fs.existsSync('uploads')) {
    fs.mkdirSync('uploads')
}

// Tipus MIME acceptats
const allowedMimeTypes = ['text/plain', 'image/jpeg', 'image/png', 'application/pdf']

// Configurar multer per guardar arxius a la carpeta "uploads"
const storage = multer.diskStorage({
    destination: (req, file, cb) => {
        cb(null, 'uploads') // Carpeta on es guarden els fitxers
    },
    filename: (req, file, cb) => {
        cb(null, Date.now() + '-' + file.originalname) // Prefixar amb un timestamp per evitar col·lisions
    }
})

// Configuració de multer amb límit de mida i validació
const upload = multer({
    storage: storage,
    limits: { fileSize: 50 * 1024 * 1024 }, // Límit de 50 MB per fitxer
    fileFilter: (req, file, cb) => {
        if (allowedMimeTypes.includes(file.mimetype)) {
            cb(null, true) // Acceptar el fitxer
        } else {
            cb(new Error('Tipus de fitxer no permès'), false) // Rebutjar el fitxer
        }
    }
})

app.post('/upload', upload.array('files', 10), (req, res) => {
    try {
        // Obtenir l'objecte JSON
        const jsonData = JSON.parse(req.body.json)

        // Obtenir els arxius
        const files = req.files.map(file => ({
            originalName: file.originalname,
            mimeType: file.mimetype,
            size: file.size,
            path: file.path
        }))

        res.json({
            message: 'Dades rebudes',
            jsonData: jsonData,
            files: files
        })
    } catch (error) {
        res.status(400).json({ error: 'Error processant la petició', details: error.message })
    }
})

app.listen(3000, () => {
    console.log('Servidor escoltant al port 3000')
})
```
