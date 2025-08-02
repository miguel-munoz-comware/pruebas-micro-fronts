Microfrontends con Next.js 15 + Module Federation + Shared Types

Este repositorio es un ejemplo completo de arquitectura de microfrontends usando:

    Next.js 15 (Host y Remotes)

    Webpack Module Federation para compartir componentes entre aplicaciones

    Librería de tipos compartida (shared-types) para mantener tipos TypeScript centralizados

    Entornos de desarrollo y producción separados mediante variables .env

📦 Estructura del proyecto

microfrontends/
│
├─ host/                 # Next.js 15 (Host principal)
│   ├─ app/
│   └─ next.config.js
│
├─ remote1/              # Microfrontend remoto 1
│   └─ src/components/
│
├─ remote2/              # Microfrontend remoto 2
│   └─ src/components/
│
└─ shared-types/         # Librería centralizada de tipos TypeScript
    ├─ src/
    │   ├─ remote1/index.ts
    │   ├─ remote2/index.ts
    │   └─ index.ts
    ├─ tsconfig.json
    └─ package.json

📖 Descripción de shared-types

shared-types es una librería interna que centraliza interfaces y tipos TypeScript para todos los microfrontends.
Ventajas

    Un solo origen de la verdad para props y tipos compartidos.

    Evita .d.ts manuales en el host y remotes.

    Compatible con desarrollo local (npm link) y producción.

Ejemplo de tipos

// shared-types/src/remote1/index.ts
export interface ButtonProps {
  label: string;
  onClick?: () => void;
}

// shared-types/src/remote2/index.ts
export interface ModalProps {
  open: boolean;
  onClose: () => void;
}

En un Remote:

import { ButtonProps } from '@company/shared-types/remote1';

En el Host:

import dynamic from 'next/dynamic';
import { ButtonProps } from '@company/shared-types/remote1';

const RemoteButton = dynamic(() => import('remote1/Button'), { ssr: false });

export default function LoginPage() {
  const handleClick: ButtonProps['onClick'] = () => alert('Clicked!');
  return <RemoteButton label="Login" onClick={handleClick} />;
}

⚙️ Configuración de entornos
.env.development

REMOTE1_URL=http://localhost:3001/_next/static/chunks/remoteEntry.js
REMOTE2_URL=http://localhost:3002/_next/static/chunks/remoteEntry.js

.env.production

REMOTE1_URL=https://cdn.miempresa.com/remote1/remoteEntry.js
REMOTE2_URL=https://cdn.miempresa.com/remote2/remoteEntry.js

    Desarrollo → carga remotes desde localhost.

    Producción → carga remotes desde URLs en CDN/servidor.

🚀 Pasos para correr el proyecto localmente
1️⃣ Instalar dependencias

# En cada proyecto
cd shared-types && npm install
cd ../host && npm install
cd ../remote1 && npm install
cd ../remote2 && npm install

2️⃣ Compilar la librería de tipos

cd shared-types
npm run build

Esto generará los .d.ts en shared-types/dist.
3️⃣ Linkear la librería localmente

Para usar los tipos sin publicarlos:

cd shared-types
npm link

cd ../host
npm link @company/shared-types

cd ../remote1
npm link @company/shared-types

cd ../remote2
npm link @company/shared-types

4️⃣ Levantar el entorno local de desarrollo

En paralelo:

npx concurrently "cd remote1 && npm run dev" "cd remote2 && npm run dev" "cd host && npm run dev"

Luego abre el host en:

http://localhost:3000

    Hot reload de componentes y tipos gracias a npm link.

    Las rutas de los remotes se cargan desde localhost:3001 y localhost:3002.

🌐 Entorno de desarrollo

    Usa .env.development automáticamente con npm run dev.

    Los remotes y host corren en local con HMR.

    Ideal para pruebas de integración de microfrontends.

🏗️ Build de producción

    Generar tipos (una sola vez si no cambian)

cd shared-types
npm run build

Build de cada app

cd host && npm run build
cd ../remote1 && npm run build
cd ../remote2 && npm run build

Levantar host en producción

    cd host
    npm run start

    El host usará las URLs configuradas en .env.production.

    Los remotes pueden desplegarse en un CDN o en servidores independientes.

✅ Comandos útiles
Acción	Comando
Instalar dependencias	npm install en cada carpeta
Compilar tipos	cd shared-types && npm run build
Linkear tipos localmente	npm link en shared-types y luego npm link @company/shared-types
Levantar todo local	npx concurrently "cd remote1 && npm run dev" "cd remote2 && npm run dev" "cd host && npm run dev"
Build de producción	npm run build en cada app
Iniciar host en producción	npm run start en host

Este README documenta el ciclo completo:

    Local → con npm link y hot reload de tipos.

    Desarrollo → usando .env.development.

    Producción → con build y .env.production.