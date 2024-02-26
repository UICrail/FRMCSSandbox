# Introduction

Some applications are foreseen to require some form of "proof of life" from the OB FRMCS and possibly vice-versa.
The present document investigates some of the scenarios.

# OB FRMCS restart

## Pre-condition workflow

```mermaid

sequenceDiagram
participant A as EVC
participant O as OB FRMCS
participant M as MCX
participant T as TS GW
participant B1 as RBC1

par OB setup
	Note over O: Start of Operation 
	Note over A,O: OBapp local binding
	Note over O,M:	MC registration
and TS setup
	Note over B1,T: TSapp local binding
	Note over T,M:	MC registration
end
```

## OFr.a Application has not initiated a session prior

```mermaid

sequenceDiagram
participant A as EVC
participant O as OB FRMCS

opt with OB PoL
  A->> O: trigger ProofOfLife endpoint
  O->>A: 200 OK
  O-->>A:  SSE POL
  alt OB FRMCS restart so POL not received by App
    Note over O:  restart
    Note over A:  App not getting POL
    %O--xA: SSE POL
    Note over A,O:  local binding
    A->> O: trigger ProofOfLife endpoint
    O->>A: 200 OK
  else OB FRMCS restart, App tries something before expected POL
    Note over O:  restart
    Note over A,O: open session attempt (ex.)
    A->>O: open session
    O->>A:  401
    Note over A,O: local binding
  end
end

opt without OB PoL
  Note over O:  restart
  A->>O: open session
  O->>A:  401
  Note over A: local binding
end  
```

## OFr.b Application has initiated a session prior

### OFr.b.1 Application does some network traffic on S1

```mermaid

sequenceDiagram
participant A as EVC
participant O as OB FRMCS

Note over A,O:  opened session S1 

opt with OB PoL
  A->> O: trigger ProofOfLife endpoint
  O->>A: 200 OK
  O-->>A:  SSE POL
  alt OB FRMCS restart so POL not received by App
    Note over O:  restart
    Note over A,O: no traffic → or ←
    Note over A:  App not getting POL
    Note over A,O:  local binding
    A->> O: trigger ProofOfLife endpoint
    O->>A: 200 OK
  else OB FRMCS restart, App tries something before expected POL
    Note over O:  restart
    Note over A,O: traffic →
    Note over O: behaviour?
  end
end

opt without OB PoL
  Note over O:  restart
  Note over A,O: traffic →
  Note over O: behaviour?
end  
```
### OFr.b.2 Application tries to open a new session S2

```mermaid

sequenceDiagram
participant A as EVC
participant O as OB FRMCS

Note over A,O:  opened session S1 

opt with OB PoL
  A->> O: trigger ProofOfLife endpoint
  O->>A: 200 OK
  O-->>A:  SSE POL
  alt OB FRMCS restart so POL not received by App
    Note over O:  restart
    Note over A,O: no service attempt
    Note over A:  App not getting POL
    Note over A,O:  local binding
    A->> O: trigger ProofOfLife endpoint
    O->>A: 200 OK
  else OB FRMCS restart, App tries something before expected POL
    Note over O:  restart
    A->>O: open session
    O->>A:  401
    Note over A,O: local binding
    Note over O: close S1
  end
end

opt without OB PoL
  Note over O:  restart
  A->>O: open session
  O->>A:  401
  Note over A,O: local binding
  Note over O: close S1
end  
```

## OFr.c Application profile allows incoming session

```mermaid

sequenceDiagram
participant A as EVC
participant O as OB FRMCS

opt with OB PoL
  A->> O: trigger ProofOfLife endpoint
  O->>A: 200 OK
  O-->>A:  SSE POL
  alt OB FRMCS restart, POL --x App but interval < restart + LB + MC
    Note over O:  restart
    Note over A:  App not getting POL
    Note over A,O:  local binding
    Note over O: MC registration
    A->> O: trigger ProofOfLife endpoint
    O->>A: 200 OK
    O-->>A: SSE incoming session
    Note over A,O: incoming session acceptance
  else OB FRMCS restart, incoming session before POL
    Note over O:  restart
    O-->>A: SSE incoming session
    Note over O: incoming session timer expires
    Note over O: DECLINE invite →
    Note over A:  App not getting POL
    Note over A,O:  local binding
    A->> O: trigger ProofOfLife endpoint
    O->>A: 200 OK
  end
end

opt without OB PoL
  alt OB FRMCS restart, MC registration
    Note over O: restart
    Note over O: MC registration
    Note over O: receipt of incoming session ←
    Note over O: app not locally bound →
    Note over A: App has no "knowledge" of service attempt
  else OB FRMCS restart, no MC registration in time
   Note over O: restart
   Note right of O: incoming session attempt ←
   Note over A,O: neither have "knowledge" of service attempt
  end
end  
```

# App restart

## Pre-condition workflow

```mermaid

sequenceDiagram
participant A as EVC
participant O as OB FRMCS
participant M as MCX
participant T as TS GW
participant B1 as RBC1

par OB setup
	Note over O: Start of Operation 
	Note over A,O: OBapp local binding
	Note over O,M:	MC registration
and TS setup
	Note over B1,T: TSapp local binding
	Note over T,M:	MC registration
end
```

## APPr.a Application had not initiated a session prior

```mermaid

sequenceDiagram
participant A as EVC
participant O as OB FRMCS

opt with App PoL
  loop every PoL interval
    A->> O: trigger ProofOfLife endpoint
    O->>A: 200 OK
  end
  alt App restart, POL --x OB 
    Note over A:  restart
    Note over O:  OB not getting POL before interval
    Note over O:  invalidate LB
    Note over A,O:  local binding
    Note over A,O:  normal flow
  else App restart, before POL
    Note over A:  restart, before POL
    Note over A,O:  local binding
    Note over O:  invalidate LB
    Note over A,O:  normal flow
  end
end

opt without App PoL
  Note over A:  restart
  Note over A,O:  local binding
  Note over O:  invalidate LB
  Note over A,O:  normal flow
end

```

## APPr.b Application had initiated a session prior

```mermaid

sequenceDiagram
participant A as EVC
participant O as OB FRMCS

Note over A,O:  opened session S1

opt with App PoL
  loop every PoL interval
    A->> O: trigger ProofOfLife endpoint
    O->>A: 200 OK
  end

  alt App restart, POL --x OB 
    Note over A:  restart
    Note over O:  OB not getting POL before interval
    Note over O:  invalidate LB, S1
    Note over A,O:  local binding
    Note over A,O:  normal flow
  else App restart, traffic ← before POL
    Note over A: restart
    Note over O: traffic ←
    Note over O: behaviour?
    Note over O:  OB not getting POL before interval
    Note over O:  invalidate LB, S1
    Note over A,O:  local binding
    Note over A,O:  normal flow
  end
end

opt without App PoL
  alt App restart, traffic ← before LB
    Note over A:  restart
    Note over O: traffic ←
    Note over O: behaviour?
    Note over A,O:  local binding
    Note over O:  invalidate LB, S1
    Note over A,O:  normal flow
  else App restart, traffic ← before LB
    Note over A:  restart
    Note over A,O:  local binding
    Note over O:  invalidate LB, S1
    Note over A,O:  normal flow
  end
end

```

## APPr.c Application profile allows incoming session


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
