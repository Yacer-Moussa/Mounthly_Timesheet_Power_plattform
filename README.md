# 📊 Monatliche Zeiterfassungs-Aggregation mit Power Automate

## 📌 Projektübersicht

Dieses Projekt automatisiert die monatliche Auswertung von Zeiterfassungsdaten aus SharePoint und speichert die aggregierten Ergebnisse in einer Excel-Datei mithilfe von **Power Automate**.

Der Flow wird automatisch am ersten Tag jedes Monats ausgeführt, liest alle Zeiterfassungseinträge des Vormonats aus, gruppiert sie nach Projekten, summiert die Vor-Ort- und Remote-Arbeitsstunden und schreibt die Ergebnisse in eine strukturierte Excel-Tabelle.

---

# 🛠️ Verwendete Technologien

| Tool                    | Zweck                              |
| ----------------------- | ---------------------------------- |
| Power Automate          | Workflow-Automatisierung           |
| SharePoint              | Quelldaten (Timesheet_List)        |
| Excel Online (OneDrive) | Speicherung der aggregierten Daten |

---

# 📁 Datenquellen

## SharePoint-Liste: `Timesheet_List`

| Spalte       | Typ   | Beschreibung                     |
| ------------ | ----- | -------------------------------- |
| Datum        | Datum | Datum des Zeiterfassungseintrags |
| Mitarbeiter  | Text  | Name des Mitarbeiters            |
| ProjectName  | Text  | Zugeordnetes Projekt             |
| OnSite_Hours | Zahl  | Vor-Ort-Arbeitsstunden           |
| Remote_Hours | Zahl  | Remote-Arbeitsstunden            |


<img width="1521" height="736" alt="image" src="https://github.com/user-attachments/assets/efe8f9ef-a5f0-48c2-8a49-67318598579f" />


## Excel-Ausgabe: `Timesheet_Aggregated.xlsx`

**Tabelle:** `timesheet_Aggregated`

| Spalte       | Typ  | Beschreibung                    |
| ------------ | ---- | ------------------------------- |
| Month        | Text | Monatsname (z. B. Mai)          |
| Year         | Zahl | Jahr (z. B. 2026)               |
| ProjectName  | Text | Projektname                     |
| OnSite_Hours | Zahl | Summe der Vor-Ort-Stunden       |
| Remote_Hours | Zahl | Summe der Remote-Stunden        |
| Total_Hours  | Zahl | Gesamtstunden (OnSite + Remote) |

---
<img width="710" height="463" alt="image" src="https://github.com/user-attachments/assets/90753186-ff9c-47cf-a28f-2bff9ed55de7" />



# ⚙️ Aufbau des Flows

```text
Recurrence (Monatlich)
├── Elemente abrufen
├── Select
├── Compose (union)
├── Variable initialisieren (varOnsite)
├── Variable initialisieren (varRemote)
└── Für jedes Projekt
    ├── Array filtern
    └── Für jede Zeile
        ├── varOnsite erhöhen
        └── varRemote erhöhen
    ├── Gesamtstunden berechnen
    ├── Zeile in Excel-Tabelle hinzufügen
    ├── varOnsite zurücksetzen
    └── varRemote zurücksetzen
```

---

# 🔄 Ablauf Schritt für Schritt

## 1. Recurrence – Trigger

Der Flow startet automatisch am **1. Tag jedes Monats um 01:00 Uhr**.

---

## 2. Elemente abrufen (SharePoint)

Ruft alle Einträge aus der Liste `Timesheet_List` ab, die zum Vormonat gehören.

### Verwendeter Filter

```text
Datum ge '@{startOfMonth(addToTime(utcNow(),-1,''Month''))}'
and
Datum lt '@{startOfMonth(utcNow())}'
```

### Beispiel

Wenn das aktuelle Datum der **01.06.2026** ist, werden alle Einträge vom Zeitraum **01.05.2026 bis 31.05.2026** geladen.

<img width="1078" height="736" alt="image" src="https://github.com/user-attachments/assets/cb9c161d-b629-4dd3-9272-aa30a0ec5e5d" />

---

## 3. Select – Projektnamen extrahieren

Extrahiert ausschließlich die Spalte `ProjectName` aus den geladenen Datensätzen.

### Beispielausgabe

```json
[
  "Solar Plant A",
  "Wind Farm B",
  "Solar Plant A",
  "Solar Plant A",
  "Wind Farm B"
]
```
---

## 4. Compose – Duplikate entfernen

Verwendet die Funktion `union()`, um doppelte Projektnamen zu entfernen.

### Ausdruck

```text
union(body('Select'), body('Select'))
```

### Ergebnis

```json
[
  "Solar Plant A",
  "Wind Farm B"
]
```

---

## 5. Variablen initialisieren

Vor Beginn der Schleifen werden zwei Variablen angelegt.

| Variable  | Typ   | Startwert |
| --------- | ----- | --------- |
| varOnsite | Float | 0         |
| varRemote | Float | 0         |


---

## 6. Apply to each – Äußere Schleife

Durchläuft alle eindeutigen Projekte aus dem vorherigen Schritt.

> 📸 *Screenshot: Äußere Apply-to-each-Schleife*

---

## 7. Array filtern

Filtert innerhalb der Schleife alle Datensätze, die zum aktuell verarbeiteten Projekt gehören.

### Bedingung

```text
item()?['ProjectName']
ist gleich
items('Apply_to_each')?['ProjectName']
```

### Beispiel

| Datum      | ProjectName   | OnSite | Remote |
| ---------- | ------------- | ------ | ------ |
| 05.05.2026 | Solar Plant A | 4      | 2      |
| 12.05.2026 | Solar Plant A | 3      | 5      |
| 20.05.2026 | Solar Plant A | 2      | 2      |


---

## 8. Apply to each – Innere Schleife

Durchläuft jede gefilterte Zeile und addiert die Arbeitsstunden.

### Vor-Ort-Stunden

```text
items('Apply_to_each_2')?['OnSite_Hours']
```

### Remote-Stunden

```text
items('Apply_to_each_2')?['Remote_Hours']
```

### Beispiel

```text
Nach Zeile 1:
varOnsite = 4
varRemote = 2

Nach Zeile 2:
varOnsite = 7
varRemote = 7

Nach Zeile 3:
varOnsite = 9
varRemote = 9
```

---

## 9. Gesamtstunden berechnen

Nach Abschluss der inneren Schleife werden die Gesamtstunden berechnet.

### Ausdruck

```text
add(variables('varOnsite'), variables('varRemote'))
```

### Beispiel

```text
9 + 9 = 18
```

<img width="918" height="740" alt="image" src="https://github.com/user-attachments/assets/7f1a0401-8184-4df3-aa23-b97d8aad6f65" />


---

## 10. Zeile in Excel-Tabelle hinzufügen

Schreibt die aggregierten Projektdaten in die Tabelle `timesheet_Aggregated`.

| Feld         | Ausdruck                                                |
| ------------ | ------------------------------------------------------- |
| Month        | `formatDateTime(addToTime(utcNow(),-1,'Month'),'MMMM')` |
| Year         | `formatDateTime(addToTime(utcNow(),-1,'Month'),'yyyy')` |
| ProjectName  | `items('Apply_to_each')?['ProjectName']`                |
| OnSite_Hours | `variables('varOnsite')`                                |
| Remote_Hours | `variables('varRemote')`                                |
| Total_Hours  | `outputs('Compose_Total')`                              |


---

## 11. Variablen zurücksetzen

Nach dem Schreiben in Excel werden beide Variablen wieder auf 0 gesetzt.

```text
varOnsite = 0
varRemote = 0
```

⚠️ **Wichtig:** Ohne das Zurücksetzen würden die Stunden des nächsten Projekts auf die bereits berechneten Werte aufaddiert werden.

---

# 📊 Beispielergebnis

## SharePoint-Daten (Mai 2026)

| Datum      | ProjectName   | OnSite | Remote |
| ---------- | ------------- | ------ | ------ |
| 05.05.2026 | Solar Plant A | 4      | 2      |
| 08.05.2026 | Wind Farm B   | 6      | 0      |
| 12.05.2026 | Solar Plant A | 3      | 5      |
| 20.05.2026 | Solar Plant A | 2      | 2      |
| 22.05.2026 | Wind Farm B   | 8      | 0      |

## Ergebnis in Excel

| Month | Year | ProjectName   | OnSite_Hours | Remote_Hours | Total_Hours |
| ----- | ---- | ------------- | ------------ | ------------ | ----------- |
| Mai   | 2026 | Solar Plant A | 9            | 9            | 18          |
| Mai   | 2026 | Wind Farm B   | 14           | 0            | 14          |

---

# 🚀 Ausführung

Der Flow läuft automatisch am ersten Tag jedes Monats.

Für eine manuelle Ausführung:

1. Power Automate öffnen
2. Flow auswählen
3. Auf **Ausführen (Run)** klicken

---

# 📝 Hinweise

* Der Flow verarbeitet immer den **Vormonat**, niemals den aktuellen Monat.
* Neue Projekte werden automatisch berücksichtigt.
* Vor der ersten Ausführung müssen die Datei `Timesheet_Aggregated.xlsx` sowie die Tabelle `tbl_Aggregated` bereits vorhanden sein.

---

