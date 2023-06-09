# Laboratorium_5

Paweł Jabłoniec

Sprawozdanie - laboratorium 5

---

### 1. Struktura aplikacji

#### 1. Dockerfile

```dockerfile
#1. BUDOWANIE APLIKACJI
# Wykorzystanie obrazu node w wersji 16-alpine jako bazowego - do budowania
FROM node:16-alpine as builder

# ARG: Parametr APP_VERSION używany do przekazania wersji obrazu
ARG APP_VERSION
# ENV: Definicja zmiennej środowiskowej APP_VERSION i przypisanie wartości z parametru APP_VERSION
ENV APP_VERSION=${APP_VERSION}

# Ustawienie katalogu roboczego na "/app"
WORKDIR /app

# Skopiowanie plików package.json i package-lock.json do kontenera
COPY my-app/package*.json ./

# Instalacja zależności przy użyciu polecenia "npm ci"
# Wykorzystałem to polecenie by przyspieszyć proces budowania obrazu
RUN npm ci

# Skopiowanie pozostałych plików aplikacji do kontenera
COPY my-app/public ./public
COPY my-app/src ./src

# Zbudowanie aplikacji przy użyciu polecenia "npm run build"
RUN npm run build




#2. WYSTAWIENIE APLIKACJI

# Wykorzystanie obrazu apache w wersji 2.4-alpine
FROM httpd:2.4-alpine

# Przekazanie parametru APP_VERSION i ustawienie zmiennej środowiskowej APP_VERSION
ARG APP_VERSION
ENV APP_VERSION=${APP_VERSION}

# Skopiowanie zbudowanej aplikacji do katalogu "/usr/local/apache2/htdocs" w obrazie apache
COPY --from=builder /app/build /usr/local/apache2/htdocs

# Wystawienie aplikacji na porcie 80
EXPOSE 80

# Uruchomienie serwera apache w trybie FOREGROUND
CMD ["httpd", "-D", "FOREGROUND"]

```

#### 2. App.js

```js
import "./App.css";
export const ver = process.env.APP_VERSION;
function App() {
  // jeżeli adresem ip będzie http://localhost to zamień go na 192.168.0.1
  const origin = window.location.origin;
  const address = origin == "http://localhost" ? "192.168.0.1" : origin;


  return (
    <div className="App">
      <header className="App-header">
        <h1>Paweł Jabłoniec</h1>
        <p>Hostname: {window.location.hostname}</p>
        <p>Address: {address}</p>
        <p>Version: {ver}</p>
      </header>
    </div>
  );
}

export default App;
```

---

### 2. Uruchomienie aplikacji

#### 1. Zbudowanie obrazu

`docker build --no-cache -t lab5:v3.0 --build-arg APP_VERSION=3.0 . `


#### 2. Uruchomienie kontenera na bazie utworzonego obrazu

`docker run -e APP_VERSION=3.0 -d -p 80:80 --name lab5v3.0 lab5:v3.0`


#### 3. Diagnostyka

`docker logs lab5v3.0`

