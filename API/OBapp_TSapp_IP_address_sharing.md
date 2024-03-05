# Introduction

The present page provides some experimental contents pertaining with exposing the IP of the trackside application end point to the Onboard application.


# ETCS flow

## connection to RBC

```mermaid

sequenceDiagram
participant A as EVC
participant O as OB FRMCS
participant M as MCX
participant T as TS GW
participant TD as TS DNS GW
participant B as RBC

Note over O: Start of Operation
par App Local Binding and subscriptions to general events
	%Note over LCAppOB,OBfrmcs: OBapp local binding
	A->>O: POST /registrations
	O->>A: 200 OK {DynamicID}
	A->>O: GET /notifications/{DynamicID}/general
	O->>A: 200 OK
	B->>T: POST /registrations
	T->>B: 200 OK {DynamicID}
	B->>T: GET /notifications/{DynamicID}/general
	T->>B: 200 OK
and MC registration for App
	Note over O,M: MCregistration
	O-->>A: SSE FSD_AVL
	Note over T,M: MCregistration
end

%open session
A->>O:	POST /sessions/{DynamicID}
par Initial answer
	O->>A:	200 OK [initial answer] (SessionID=S1)
and Session establishment
	O->>M:  MCData IPcon P2P request
  M->>T:  MCData IPcon P2P request
	T-->>B:	SSE Incoming Session (SessionID=S3)
	B->>T:	PUT /sessions/{DynamicID}/S3 (appDestIP=RBC IP)
	T->>B: 200 OK
  par MCData answer
    T->>M: MCData IPcon P2P response
    M->>O: MCData IPcon P2P response
  and DNS update
    Note over T,TD:  update DNS (({DynamicID},S3) ↔ RBC IP)
  end
  Note over O,TD:  DNS query
end
O-->>A:	SSE Final answer (SessionID=S1,appDestIP=RBC IP)	
Note over A,B: establishment of EVC ↔ RBC connection

```


