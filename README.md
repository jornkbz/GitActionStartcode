## Ejercicio 01
1. Se crea el repositorio
2. Se crea  una rama diferente a main (muy importante este paso)
![image](https://github.com/user-attachments/assets/11f37663-8324-40ba-98cd-ff0adc1d09e5)

A partir de ahora se trabajara en esta nueva rama para el resto de pasos:



3. Se crea el workflow, para ello se genera un fichero en .github/workflows con extension yaml y se sube al repositorio.
```
name: FrontCI

on:

  pull_request:

    branches:

      - main     # Ajusta según la(s) rama(s) donde quieras que se active

    paths:

      - '.start-code/hangman-front/**'  # Importante: solo se activa si hay cambios dentro de hangman-front

jobs:

  build-and-test:

    runs-on: ubuntu-latest

  

    steps:

      - name: Check out repository

        uses: actions/checkout@v3

  

      - name: Set up Node

        uses: actions/setup-node@v3

        with:

          node-version: 16  # O la versión que uses

  

      - name: Install dependencies

        run: npm install

        working-directory: .start-code/hangman-front

  

      - name: Build project

        run: npm run build

        working-directory: .start-code/hangman-front

  

      - name: Run tests

        run: npm test

        working-directory: .start-code/hangman-front
```

4. El workflow solo se ejecutará cuando haya cambios en el proyecto "hangman-front" y exista una nueva pull request (deben darse las dos condiciones a la vez) 
5. Una vez subidos los cambios se notificará con una pull_request la cual debemos aceptar para que se ejecute el workflow en la interfaz de gitHub del repositorio , se selecciona la rama y aparecerá un boton "contribute", se despliega y hay que seleccionar pull_request. Cuando se acepte el pull y halla cambios en el front se ejecutará el worklfow.

![image](https://github.com/user-attachments/assets/32e5c4a9-ca1e-46d7-aa85-75197b70e7d0)



## Ejercicio 2
1. Primer paso es generar el token generar el token:
![image](https://github.com/user-attachments/assets/e548ee28-b715-419d-9c03-b37d45d8f359)

package-registry


3. Tras generar el token hay que generar el secreto para vincularlo al token .Para ello hay que meter el token en el secrets del proyecto:
![image](https://github.com/user-attachments/assets/e7de2051-7c66-4c7d-9bda-717d44513241)



4. Crear el nuevo yaml y ejecutarlo manualmente.
```
name: Hangman Front CD

  

on:

  workflow_dispatch:  # Permite ejecutarlo manualmente desde la pestaña "Actions"

  

jobs:

  build-and-publish-docker:

    runs-on: ubuntu-latest

  

    steps:

      - name: Check out code

        uses: actions/checkout@v3

  

      # 1. Iniciar sesión en GitHub Container Registry

      - name: Log in to GitHub Container Registry

        uses: docker/login-action@v2

        with:

          registry: ghcr.io

          username: ${{ github.actor }}

          password: ${{ secrets.STARCODE_CI_MAUAL }}

  

      # 2. Construir la imagen Docker

      - name: Build the Docker image

        run: |

          docker build -t ghcr.io/jornkbz/gitactionstartcode/hangman-front:latest  \

          -f .start-code/hangman-front/Dockerfile \

          .start-code/hangman-front

  

      # 3. Publicar la imagen en ghcr.io

      - name: Push the Docker image

        run: |

          docker push ghcr.io/jornkbz/gitactionstartcode/hangman-front:latest
```

5. Tras la ejecución tenemos este resultado


![image](https://github.com/user-attachments/assets/4ececdda-c158-4383-bd30-0bea47cdf5db)


6. Como se puede apreciar en el apartado "Packages" se ha subido la imagen.
![image](https://github.com/user-attachments/assets/fe34a4bc-de1f-4673-acc2-2e378801d71a)



## Ejercicio 4: OPCIONAL

1. Para este ejercicio se genera un nuevo workflow de ejecución manual
```
name: E2E Tests

  

on:

  workflow_dispatch:

  

jobs:

  run-e2e-tests:

    runs-on: ubuntu-latest

    steps:

      - name: Check out repo

        uses: actions/checkout@v3

  

      # 1) Build hangman-api

      - name: Build hangman-api image

        working-directory: .start-code/hangman-api

        run: docker build -t hangman-api:latest .

  

      # 2) Build hangman-front

      - name: Build hangman-front image

        working-directory: .start-code/hangman-front

        run: docker build -t hangman-front:latest .

  

      # 3) Run the containers

      - name: Start hangman-api

        run: docker run -d --name hangman-api -p 3001:3000 hangman-api:latest

  

      - name: Start hangman-front

        run: docker run -d --name hangman-front -p 8080:8080 -e API_URL=http://localhost:3001 hangman-front:latest

  

      # (Opcional) Esperar a que inicien

      - name: Wait containers

        run: sleep 5

  

      # 4) Instalar y ejecutar los E2E

      - name: Install E2E dependencies

        working-directory: .start-code/hangman-e2e/e2e

        run: npm install

  

      - name: Run E2E

        working-directory: .start-code/hangman-e2e/e2e

        run: npx cypress run  # o "npm run test", según tu script
```

Este es el resultado tras la ejecución.

![image](https://github.com/user-attachments/assets/d98c6038-42b2-45fc-9767-b7b8583c6205)



Este es el desglose:

![image](https://github.com/user-attachments/assets/fd33c634-9079-4e9e-8af0-461eadbfe5c7)


Segunda parte del desglose:

![image](https://github.com/user-attachments/assets/8b5481f6-9b1c-4fac-8000-862e62420bcf)



