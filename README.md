``` plantuml
@startuml
    !include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Component.puml

    title Component diagram for ERP Integration Microservice (FinOps Portal API)
    
    ContainerDb(ssdb, "Starship Database", "SQL Server AWS RDS", "Booking and transaction system of record")
    Container_Ext(ssservices, "Starship Services", "SOAP/.Net API", "Booking management bundle of service endpoints")
    Container_Ext(ppservice, "Product Publisher Service", "NServiceBus/.Net API", "Product updates trigger push to queues for consuming")
    Container_Ext(bnservice, "Booking Notification Service", "NServiceBus/.Net API", "Booking updates trigger push to queues for consuming")
    Container_Ext(deloitte, "Deloitte Integration\n🥷", "Black Box", "Automagical data transformation and ingestion into D365")
    System_Ext(newrelic, "New Relic", "Application performance monitoring platform SaaS")

    Container_Boundary(finops, "FinOps Portal") {
        Component_Ext(portal, "Finance Portal", "Blazer/.Net App", "Finance data sync facilitation web app")
        Boundary(erpapi, "ERP Integration Microservice", "COMPONENT") {
            Component(api, "Backend For Frontend", ".Net API", "Handles UI interactions")
            Component(listener, "NServiceBus Listener", "NServiceBus Handler", "Subscribes to queue and consumes finance updates as JSON payloads")
            Component(database, "DAO Service", "Sql Data Reader", "Retrieves transaction details from Starship DB")
            Component(d365, "D365 Update Handler", "Batch Data API", "Pushes batch updates into D365")
            Component(logger, "Logger", "SEQ Sink Log Forwarder", "Uses SEQ agent to forward logs onto New Relic")

            Rel(api, database, "Updates to sync config and transaction Agent IDs")
            Rel(api, listener, "Update which transaction agents to listen for")
            Rel(listener, database, "On update, fetch transaction<br/>records from Starship")
            Rel(database, d365, "After transaction fetch<br/>send records to D365")
            Rel(api, logger, "Log activity")
            Rel(listener, logger, "Log activity")
            Rel(database, logger, "Log activity")
            Rel(d365, logger, "Log activity")
        }
    }
    
    Rel(portal, api, "UI requests", "JSON/HTTPS")
    Rel_Up(ssdb, ppservice, "Product update triggers notification", "SOAP/HTTPS")
    Rel_Up(ssdb, bnservice, "Booking update triggers notification", "SOAP/HTTPS")
    Rel(listener, ssservices, "Request finance booking data", "SOAP/HTTPS")
    Rel(listener, ppservice, "Pulls notifications off queue", "JSON/HTTPS")
    Rel(listener, bnservice, "Pulls notifications off queue", "JSON/HTTPS")
    Rel(database, ssdb, "Updated Payments, Agents", "SQL/TCP")
    Rel(ssservices, ssdb, "Retrieve finance booking data", "SQL/TCP")
    Rel(d365, deloitte, "Batch records and send to Azure queue which Deloitte integration listens to", "JSON/HTTPS")
    Rel(logger, newrelic, "Forward application events and system logs", "JSON/HTTPS")

@enduml
```