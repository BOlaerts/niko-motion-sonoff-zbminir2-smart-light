# Home Assistant Automatisatie

## Doel

Deze automatisatie regelt de samenwerking tussen:

- beweging (via fysieke input op de Sonoff)
- manuele bediening (knoppen in Home Assistant)
- detach modus (manuele override)
- timer helper (automatische uitschakeling)

Het doel is:

- **Beweging kan het licht aanzetten**
- **Manuele bediening kan het licht “overnemen”** (detach aan)
- **Manueel uitschakelen kan enkel in manuele modus** (detach aan)
- **Na timer-afloop schakelt het licht uit**

---

## Entiteiten

| Entity | Functie |
|--------|-----------|
| switch.licht | Sonoff relais (licht) |
| switch.detach_relay_mode | Detach modus (manual override) |
| switch.schakelaar_licht | Fysieke schakelaar / toggle input |
| button.scene_switch | Scene/toggle knop |
| timer.licht_timer | Auto-off timer helper |

---

## Gedrag

### A) Manuele toggle (scene_switch of schakelaar_licht)

**1) Licht UIT → manuele modus starten**
- detach modus **AAN**
- licht **AAN**
- timer **start**

**2) Licht AAN + detach AAN → manueel uitschakelen**
- detach modus **UIT**
- licht **UIT**
- timer **stop**

**3) Licht AAN + detach UIT → “manuele overname”**
Dit is het belangrijke bijstuurgedrag:

- Als het licht aan staat door **beweging** (detach UIT),
  dan zorgt een druk op de knop **niet** voor uitschakelen.
- In plaats daarvan zetten we detach modus **AAN** en starten (of resetten) we de timer.
- Het licht blijft dus aan, maar staat nu in **manuele modus**.

Hiermee vermijd je het scenario waarbij je het licht manueel uitzet terwijl de bewegingsmelder nog actief is
(en het licht daarna opnieuw aan springt).

---

### B) Automatische timerlogica (zonder dubbele starts)

- **Manuele toggle** (knoppen) start/stop de timer expliciet in de toggle-acties.
- **Motion-mode** (detach UIT) start/stop de timer enkel via het aan/uit gaan van `switch.licht`.

Concreet:
- Licht **AAN** en **detach UIT** → timer start (motion)
- Licht **UIT** en **detach UIT** → timer stopt (motion)
- In manuele modus (detach AAN) komt timer start/stop **alleen** uit de toggle-acties.

Zo vermijd je dat de timer twee keer start wanneer een manuele toggle ook `switch.licht` naar **on** laat gaan.

---

## Configuratiebestand

👉 [Automation YAML](../home-assistant/automation.yaml)

---

## Timer gedrag (zoals gevraagd)

- **Manuele modus (detach AAN)** gebruikt de `timer.licht_timer`:
  - Timer **start** bij manueel aanzetten (of manuele overname)
  - Timer **stopt** bij manueel uitzetten
  - Timer **afgelopen** → licht UIT + detach UIT

- **Automatische modus (detach UIT)** wordt **volledig** door de bewegingsmelder gestuurd.
  - Home Assistant start/stop de timer dan niet.
  - De Niko bepaalt zelf hoelang L’ actief blijft (TIME-instelling).
