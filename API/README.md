# Introduction

The present page provides some informational contents on usage of the OBapp / TSapp API in call flows

# Canonical call flow

```mermaid

sequenceDiagram
participant A as LCApp
participant B as "OB FRMCS"

	A->>B: POST /registrations
	B->>A: 200 OK {DynamicId}

	A->>B: GET /notifications/{DynamicID}/general
	B->>A: 200 OK

	A->>B: GET /notifications/{DynamicID}/{channel}
	B->>A: 200 OK
	
	B->>A: SSE from /notifications/{DynamicId}/{channel}
	% potentially SSE on /notifications/{DynamicId}/{channel}
	
	A->>B: POST /sessions/{DynamicId}/
	B->>A: 200 OK [sessionID]
	B->>A: SSE from /notifications/{DynamicId}/general
	
	A->>B: GET /sessions/{DynamicId}/{sessionID}
	B->>A: 200 OK {sessionIfn}

	A->>B: DELETE /sessions/{DynamicId}/{sessionID}
	B->>A: 200 OK

	B->>A: SSE on /notifications/{DynamicId}/general
	A->>B: PUT on /sessions/{DynamicId}/{sessionID}

	B->>A: SSE on /notifications/{DynamicId}/general
	
	A->>B: DELETE /notifications/{DynamicId}/{channel}
	B->>A: 200 OK

	A->>B: DELETE /registrations/{DynamicId}
	B->>A: 200 OK

	B->>A: SSE on /notifications/{DynamicId}/general

```

# ETCS flow

## connection to RBC

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
	O->>A:	200 OK [initial answer]
and Session establishment
	Note over O,T: MCdata
	T-->>B:	SSE Incoming Session {SessionID}
	B->>T:	PUT /sessions/{DynamicID}/{SessionID}
	T->>B: 200 OK
end
O-->>A:	SSE Final answer {SessionID}	
Note over A,B: establishment of EVC ↔ RBC connection

```

## inter-RBC (national)

```mermaid

sequenceDiagram
participant A as EVC
participant O as OB FRMCS
participant M as MCX
participant T as TS GW
participant B1 as RBC1
participant B2 as RBC2

par OB setup
	Note over O: Start of Operation 
	Note over A,O: OBapp local binding
	Note over O,M:	MC registration
and TS setup
	Note over B1,T: TSapp local binding
	Note over B2,T: TSapp local binding
	Note over T,M:	MC registration
end

% existing session with B1
Note over A,B1: session S1 EVC ↔ RBC1

% establishment of session to B2
Note over A:	SBG {RBC2}

A->>O:	POST /sessions/{DynamicID} [RBC2]
par Initial answer
	O->>A:	200 OK [initial answer]
and Session establishment
	Note over O,T: MCdata
	T-->>B2:	SSE Incoming Session {SessionID}
	B2->>T:	PUT /sessions/{DynamicID}/{SessionID}
	T->>B2: 200 OK
end
O-->>A:	SSE Final answer {SessionID=S2}	
Note over A,B2: establishment of EVC ↔ RBC2 connection
Note over A,B2: ← end connection with RBC1
A->>O:	DELETE /sessions/{DynamicID}/S1
par ack to EVC
	O->>A:	200 OK
and closure towards TS
	Note over O,T:	MCdata termination [RBC1]
	T-->>B1: SSE S1 closure
end
Note over A,B2: → ended connection with RBC1

```

## inter-RBC (cross border)

```mermaid

sequenceDiagram
participant A as EVC
participant O as OB FRMCS
participant M1 as MCX1
participant M2 as MCX2
participant T1 as TS GW1
participant T2 as TS GW2
participant B1 as RBC1
participant B2 as RBC2

par OB setup
	Note over O: Start of Operation 
	Note over A,O: OBapp local binding
	Note over O,M1:	MC registration
and TS setup
	Note over B1,T1: TSapp local binding
	Note over B2,T2: TSapp local binding
	Note over T1,M1: MC registration
	Note over T2,M2: MC registration
end

% existing session with B1
Note over A,B1: session S1 EVC ↔ RBC1

% BXL
Note over O,M1: NTT to move to Domain 2
Note over O,M2: acquisition of Domain 2
O-->>A: SSE FSD_AVL [Domain 2]

% establishment of session to B2
Note over A:	SBG {RBC2}

A->>O:	POST /sessions/{DynamicID} [RBC2]
par Initial answer
	O->>A:	200 OK [initial answer]
and Session establishment
	Note over O,T2: MCdata
	T2-->>B2:	SSE Incoming Session {SessionID}
	B2->>T2:	PUT /sessions/{DynamicID}/{SessionID}
	T2->>B2: 200 OK
end
O-->>A:	SSE Final answer {SessionID=S2}	
Note over A,B2: establishment of EVC ↔ RBC2 connection
Note over A,B2: ← end connection with RBC1
A->>O:	DELETE /sessions/{DynamicID}/S1
par ack to EVC
	O->>A:	200 OK
and closure towards TS
	Note over O,T1:	MCdata termination [RBC1]
	T1-->>B1: SSE S1 closure
end
Note over A,B2: → ended connection with RBC1

```
