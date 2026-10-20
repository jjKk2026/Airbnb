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

### 5. Conectar Gmail (una sola vez)
1. Entra en https://console.cloud.google.com y crea un proyecto.
2. Activa la **Gmail API**.
3. En *Pantalla de consentimiento OAuth*, elige "Externo" y añade tu correo como usuario de prueba.
4. En *Credenciales*, crea un **ID de cliente OAuth** de tipo "App de escritorio" y descarga el JSON.
5. Guárdalo como `credentials.json` en esta carpeta.
6. Ejecuta `python agente.py`: se abre el navegador para que des permiso. Se guarda `token.json`.

### 6. Usarlo
```bash
python agente.py            # revisa una vez
python agente.py --cada 5   # revisa cada 5 minutos
```

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
