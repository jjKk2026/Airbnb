# Agente de mensajes de Airbnb

Lee en tu Gmail los avisos de mensajes nuevos de Airbnb, escribe la respuesta con Claude
y la deja como **borrador** en Gmail, en el mismo hilo. **Nunca envía nada solo.**

- Usa solo la información que pongas en `config/alojamientos.yaml` (tus 3 anuncios).
- Si el tema es delicado (reembolsos, cancelaciones, quejas, mascotas…) o falta información,
  pone la etiqueta **Airbnb-Revisar** en el correo.
- Los correos ya leídos por el agente quedan con la etiqueta **Airbnb-Agente**.

## Cómo funciona

```
Huésped escribe en Airbnb
   → Airbnb te manda un email
   → el agente lo lee y escribe la respuesta
   → borrador en Gmail
   → tú lo revisas y lo envías (o lo copias en la app)
```

## Puesta en marcha

### 1. Instalar
```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

### 2. Clave de Claude
Crea una clave en https://platform.claude.com y:
```bash
export ANTHROPIC_API_KEY="sk-ant-..."
```

### 3. Rellenar tus anuncios
Edita `config/alojamientos.yaml`. `nombre_en_airbnb` tiene que coincidir con el nombre del
anuncio que aparece en los correos. Cuanto más completo, menos veces te pedirá revisar.

### 4. Probar sin Gmail
```bash
python agente.py --prueba ejemplos/mensaje.txt
```

### 5. Conectar Gmail (una sola vez, en tu ordenador)
1. Entra en https://console.cloud.google.com y crea un proyecto.
2. Activa la **Gmail API**.
3. En *Pantalla de consentimiento OAuth*, elige "Externo" y añade tu correo como usuario de prueba.
4. En *Credenciales*, crea un **ID de cliente OAuth** de tipo "App de escritorio" y descarga el JSON.
5. Guárdalo como `credentials.json` en esta carpeta.
6. Ejecuta `python agente.py --conectar`: se abre el navegador para que des permiso. Se guarda `token.json`.

### 6. Usarlo en tu ordenador
```bash
python agente.py            # revisa una vez
python agente.py --cada 5   # revisa cada 5 minutos
```

## Ponerlo en GitHub (sin tener el ordenador encendido)

GitHub ejecuta el agente cada 30 minutos (`.github/workflows/agente.yml`).

1. **Haz el repo privado:** Settings → General → abajo del todo, *Change visibility* → Private.
2. **Que el permiso de Gmail no caduque:** en Google Cloud, *Pantalla de consentimiento OAuth*,
   pulsa **Publicar app** (estado "En producción"). Si se queda en "Prueba", caduca cada 7 días.
   Google te avisará de que la app no está verificada: es normal, porque solo la usas tú.
3. En tu ordenador ejecuta `python agente.py --conectar`. Se crea `token.json`.
4. En GitHub: Settings → Secrets and variables → Actions → **New repository secret**. Crea tres:
   - `ANTHROPIC_API_KEY`: tu clave de Claude.
   - `GMAIL_TOKEN`: todo el contenido de `token.json`.
   - `ALOJAMIENTOS_YAML`: todo el contenido de tu `config/alojamientos.yaml` relleno.
5. Pestaña **Actions** → *Agente de mensajes Airbnb* → **Run workflow** para probarlo.
6. Borra `token.json` de tu ordenador si no lo vas a usar allí.

## Seguridad

- **Las claves van en Secretos de GitHub:** se guardan cifradas, no aparecen en el código
  y GitHub las tapa en los registros. Nunca subas `credentials.json` ni `token.json`
  (ya están en `.gitignore`).
- **Los datos de tus anuncios van en un secreto**, no en el repo. Deja el archivo del repo
  con los datos de ejemplo.
- **El registro de GitHub no guarda mensajes de huéspedes:** solo pone "borrador creado".
- **El agente no envía nada:** solo crea borradores. Si algo falla, como mucho tendrás un borrador malo.
- **Lo que podría pasar si alguien robara el permiso de Gmail:** podría leer tu correo.
  Para reducir ese riesgo:
  - Activa la verificación en dos pasos en GitHub y en Google.
  - Mejor aún: usa un **Gmail solo para Airbnb**. Cambia el correo de tu cuenta de Airbnb a ese
    Gmail, o reenvía allí solo los correos de Airbnb.
  - Si sospechas algo, quita el permiso en https://myaccount.google.com/permissions y
    borra la clave en platform.claude.com.
- **Pon un límite de gasto** a tu clave de Claude en platform.claude.com (por ejemplo, 10 $/mes).

## ¿El borrador llega a Airbnb si lo envío?
Airbnb suele permitir responder a sus avisos por email y lo pasa al chat de la reserva.
**Pruébalo una vez con un mensaje tuyo.** Si no llega, copia el borrador y pégalo en la app de Airbnb.

## Coste aproximado
Unos céntimos por mensaje con Claude Opus 5.5. Para gastar menos, cambia `MODELO` en
`agente.py` a `claude-haiku-5-5`.

## Siguientes pasos posibles
- Aviso al móvil (Telegram/WhatsApp) con el borrador y botón "Enviar".
- Envío automático de respuestas fáciles (wifi, horarios) cuando confíes en él.
- Mensajes programados: bienvenida el día antes, instrucciones de salida, pedir reseña.
