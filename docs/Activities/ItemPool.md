---
title: Aufgaben-Pool
parent: Netzwerk-Aktivitäten
---

# Geteilter Aufgabenpool

An einen geteilten Aufgabenpool stellen sich mehrere Anforderungen:

- Austauschformat zur Abbildung von - insb. - Aufgaben zu SQL-Abfragen und ER-Modellierung
- Vollständige Information über eine Aufgabe, um diese in ein beliebiges System (z. B. LMS wie Moodle, Ilias oder OPAL; Assessment-Systeme wie Stack oder Onyx; eigens entwickelte Systeme, wie z. B. EasyTutor oder ALADIN) "zurück zu übersetzen"
- Auffindbarkeit von Aufgaben anhand sinnvoller Suchkriterien

Die [Item-Pool-API](https://github.com/neda-digital/ItemPoolAPI) versucht, diese Anforderungen vollständig zu erfüllen ohne hohen manuellen Aufwand für Import- und Export zu verursachen.

Nachfolgendes Sequenz-Diagramm zeigt die Prozessschritte der API während des Imports einer Aufgabe. Der erste Prozessschrit ist die Zerlegung einer Aufgabe in einzelne Bestandteile (Stimuli [^1] und Musterlösungen), das Erfassen von Metadaten - insbesondere Merkmale der jeweiligen Aufgabenbestandteile (z. B. Metriken zur Komplexität einer SQL-Abfrage oder Verständlichkeit eines Texts) - und das Abspeichern der Bestandteile mitsamt der erfassten Metadaten.

```mermaid!
---
config:
  look: classic
---
sequenceDiagram
  actor Client as Client
  participant TaskRegistrationRouter as TaskRegistrationRouter
  participant TaskRegistrationController as TaskRegistrationController
  participant TaskHandler as TaskHandler

  autonumber
  Client ->> TaskRegistrationRouter: POST /registerTask

  activate TaskRegistrationRouter
  TaskRegistrationRouter ->> TaskRegistrationController: register_task
  deactivate TaskRegistrationRouter

  activate TaskRegistrationController
  TaskRegistrationController ->> TaskRegistrationController: _select_task_handler
  TaskRegistrationController ->> TaskHandler: process_task
  deactivate TaskRegistrationController

  activate TaskHandler
  loop
    TaskHandler -> TaskHandler: _register_materials

    alt
        TaskHandler ->> TaskHandler: _handle_id
        TaskHandler ->> TaskMaterialRegistrationController: register_material
        deactivate TaskHandler

        activate TaskMaterialRegistrationController
        TaskMaterialRegistrationController ->> TaskMaterialRegistrationController: _select_material_handler
        TaskMaterialRegistrationController ->> TaskMaterialHandler: process_material
        deactivate TaskMaterialRegistrationController

        activate TaskMaterialHandler
        TaskMaterialHandler ->> TaskMaterialHandler: _select_metrics_handler
        TaskMaterialHandler ->> MetaDataInferenceHandler: infer_metadata
        deactivate TaskMaterialHandler

        activate MetaDataInferenceHandler
        MetaDataInferenceHandler -->> TaskMaterialHandler: {{metadata}}
        deactivate MetaDataInferenceHandler
    end

    TaskMaterialHandler -->> TaskMaterialRegistrationController: {{metadata}}
    TaskMaterialRegistrationController ->> DAO: store_task_material
    DAO -->> TaskMaterialRegistrationController: {{task_material_id}}
    TaskMaterialRegistrationController -->> TaskHandler: {{task_material_id}}
  end

  TaskHandler -->> TaskRegistrationController: {{task_material_id}}

  TaskRegistrationController ->> DAO: store_task
  DAO -->> TaskRegistrationController: {{TaskID}}

  TaskRegistrationController -->> TaskRegistrationRouter: {{TaskId}}

  TaskRegistrationRouter -->> Client: {{TaskID}}
```

Mehr Informationen finden sich unter [^2].

Die API soll zukünftig um weitere Aufgabentypen und die Erfassung von (anonymisierten) Lösungsversuchen der jeweiligen Aufgaben durch Lernende erweitert werden.

---

[^1]: Garner, W.R. (1974). The Stimulus in Information Processing. In: Moskowitz, H.R., Scharf, B., Stevens, J.C. (eds) Sensation and Measurement. Springer, Dordrecht. https://doi.org/10.1007/978-94-010-2245-3_7
[^2]: https://github.com/neda-digital/ItemPoolAPI/wiki/API-Sequence-Diagram
