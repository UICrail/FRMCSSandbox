# Introduction

The present folder serves as a repository to collaborate on call flows pertaining to E2E Procedures.

# Start of Operation

```mermaid

sequenceDiagram
participant AL as OB LC App
participant AT as OB TC App
participant O as OB FRMCS
participant FTD as FRMCS Transport Domain
participant NFTD as Non-FRMCS Transport Domain
participant FSD as FRMCS Service Domain

Note over O: FRMCS Start of Operation order

Note over O: selection of one or more OB RMs for FSoOP
Note over O, FTD: OB RM registration to 5GS [765-1]
Note over O, NFTD: [optional] OB RM registration to Non-FRMCS Transport Domain [not FRMCS-specified]

Note over O: selection of an OB RM for FRMCS Multipath discovery
Note over O, FTD: FRMCS Multipath discovery procedure [765-1]

loop per Loose Coupled Startup Application
  Note over O: assignment of IP address to MC client [765-3]
  opt if Multipath available and Multipath enabled for the application
    Note over O, FTD: selection of applicable data paths [765-1]
  end
end

Note over O, FTD: Transport path establishment for MC signaling [765-1]
O -->> AT:  FTD_AVL  [765-3]

loop per Loose Coupled Startup Application
  Note over O, FSD: MC user registration  [765-2]
  O -->> AL:  FSD_AVL [765-3]
end

Note over O: FRMCS Start of Operation complete
```

# Close of Operation

```mermaid

sequenceDiagram
participant AL as OB LC App
participant AT as OB TC App
participant O as OB FRMCS
participant FTD as FRMCS Transport Domain
participant NFTD as Non-FRMCS Transport Domain
participant FSD as FRMCS Service Domain

Note over O: FRMCS shutdown order

par cleanup network side at Service Stratum
  loop per LC in LBA state
    Note over O,FSD:  MC user deregistration [765-2]
  end
and cleanup app side
  loop per LC in LBA state
    opt if opened sessions
      loop per session S
        O -->>AL:  session closure notification <S> [765-3]
      end
    end
    O-->>AL:  FSD_NAVL [765-3]
    O-->>AL:  deregistration from OB [765-3]
  end
end

opt if Multipath
  Note over O,FTD: Multipath CP cleanup procedure [765-1]
end

par cleanup network side at Transport Stratum
  loop per OB RM
    Note over O,FTD: OB RM de-registration from 5GS [765-1]
  end
and cleanup app side
  loop per TC in LBA state
    O-->>AT:  FTD_NAVL [765-3]
    O-->>AT:  deregistration from OB [765-3]
  end
end 

Note over O: FRMCS shutdown complete
```

# Inter-FRMCS Domain Transition

```mermaid

sequenceDiagram
participant AL1 as OB LC App (no session)
participant AL2 as OB LC App (active)
participant AT as OB TC App
participant O as OB FRMCS
participant M1 as MCX1
participant M2 as MCX2
participant T1 as TS GW1
participant T2 as TS GW2
%participant B1 as RBC1
%participant B2 as RBC2

par OB setup
	Note over O: Start of Operation 
	Note over AL1,O: OBapp local binding
  	Note over AL2,O: OBapp local binding
  	Note over AL2,T1: active session S1
  	Note over O,M1:	MC user registration
and TS setup
	%Note over B1,T1: TSapp local binding
	%Note over B2,T2: TSapp local binding
	Note over T1,M1: MC user registration
	Note over T2,M2: MC user registration
end

% BXL
Note over O,M1: NTT to move to Domain 2  [765-2]
Note over O,M2: acquisition of FRMCS Transport Domain 2  [765-1]

% BXL - TC
loop Tight Coupled Application
  O-->>AT: SSE FTD_AVL [Domain 2]  [765-3]
  Note over AT,M2:  MC user migration FSD1 → FSD2 [765-2]
end

% BXL - LC
loop Type II applications
  opt applications entering the Domain of applicability
    Note over O,M2: MC user registration  [765-2]
    O-->>AL1: SSE FSD_AVL [Domain 2]  [765-3]  
  end
  opt applications exiting the Domain of applicability
    Note over O,M1: MCX session/transaction termination [765-2]
    O-->>AL2:  SSE session closure (S1) [765-3]
    Note over O,M1: MC user deregistration [765-2]
    O-->>AL2:  SSE FSD_NAVL [Domain 1]  [765-3]
  end
end

loop Type I or Type III applications
  opt no open session
    Note over O,M2: MC user migration FSD1 → FSD2 [765-2]
    O-->>AL1: SSE FSD_AVL [Domain 2]  [765-3]
  end
  opt at least one open session
    Note over O,M2: MC user migration FSD1 → FSD2 [765-2]
    O-->>AL1: SSE FSD_AVL [Domain 2]  [765-3]
    Note over O,T2:  ongoing MCX session/transaction transfer FSD1 → FSD2? [765-2]
    Note over O,T1: MCX session/transaction termination? [765-2]  
  end
end

```

