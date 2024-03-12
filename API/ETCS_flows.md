# Introduction

The present page provides an illustration of the OBapp / TSapp API in call flows in the context of ETCS.

# ETCS flow

## connection to RBC (high-level)

```mermaid

sequenceDiagram
participant A as EVC
participant O as OB FRMCS
participant M as MCX
participant T as TS GW
participant B as RBC

Note over O: Start of Operation
par App Local Binding and subscriptions to general events
	%Note over LCAppOB,OBfrmcs: OBapp local binding
	A->>O: POST /registrations
	O->>A: 200 OK (DynamicID=D1)
	A->>O: GET /notifications/{DynamicID}/general
	O->>A: 200 OK
	B->>T: POST /registrations
	T->>B: 200 OK (DynamicID=D2)
	B->>T: GET /notifications/{DynamicID}/general
	T->>B: 200 OK
and MC registration for App
	Note over O,M: MCregistration
	O-->>A: SSE FSD_AVL
	Note over T,M: MCregistration
end

%open session
A->>O:	POST /sessions/{DynamicID=D1} (localAppIPAddress=IPe,recipients=[RBC])
par Initial answer
	O->>A:	200 OK [initial answer] (SessionID=S1)
and Session establishment
	Note over O,T: MCdata
	T-->>B:	SSE Incoming Session (SessionID=S3)
	B->>T:	PUT /sessions/{DynamicID}/{SessionID=S3} (localAppIPAddress=IPr)
	T->>B: 200 OK
end
O-->>A:	SSE Final answer (SessionID=S1)	
Note over A,B: establishment of EVC ↔ RBC connection between IPe and IPr

```

## connection to RBC (details)

```mermaid

sequenceDiagram
participant A as EVC
participant O as OB FRMCS
participant M as MCX
participant T as TS GW
participant B as RBC

Note over O: Start of Operation
par App Local Binding and subscriptions to general events
	%Note over LCAppOB,OBfrmcs: OBapp local binding
	A->>O: POST /registrations
	O->>A: 200 OK (DynamicID=D1)
	A->>O: GET /notifications/{DynamicID}/general
	O->>A: 200 OK
	B->>T: POST /registrations
	T->>B: 200 OK (DynamicID=D2)
	B->>T: GET /notifications/{DynamicID}/general
	T->>B: 200 OK
and MC registration for App
	Note over O,M: MCregistration
	O-->>A: SSE FSD_AVL
	Note over T,M: MCregistration
end

%open session
A->>O:	POST /sessions/{DynamicID=D1} (localAppIPAddress=IPe,recipients=[RBC])
par Initial answer
	O->>A:	200 OK [initial answer] (SessionID=S1)
and Session establishment
  O->>M:  MCData IPcon P2P request
  M->>T:  MCData IPcon P2P request
	T-->>B:	SSE Incoming Session (SessionID=S3)
	B->>T:	PUT /sessions/{DynamicID}/{SessionID=S3} (localAppIPAddress=IPr)
	T->>B: 200 OK
  T->>M: MCData IPcon P2P response (AnyExt(remoteAppIPAddress=IPr))
  M->>O: MCData IPcon P2P response (AnyExt(remoteAppIPAddress=IPr))
end
O-->>A:	SSE Final answer (SessionID=S1, remoteAppIPAddress=IPr)	

Note over A,B: establishment of EVC ↔ RBC connection between IPe and IPr
Note over A,B: TCP establishment
A->>B:  SYN
B->>A:  SYN ACK
A->>B:  ACK
Note over A,B: TLS establishment
A->>B: Client Hello
B->>A: Server Hello
A->>B: Client Finished
B->>A: Server Finished
Note over A,B: ALE
A->>B:  AU1
B->>A:  AU2
```

