# Introduction

The present page provides some informational contents on usage of the OBapp / TSapp API in call flows

# Canonical call flow

```mermaid

sequenceDiagram
participant A as LCApp
participant B as "OB FRMCS"

	A->>B: GET /notifications/general
	B->>A: 200 OK
	
	A->>B: POST /registrations
	B->>A: 200 OK {DynamicId}

	A->>B: POST /notifications/{channel}
	B->>A: 200 OK
	% could also envisage GET /notifications/{DynamicId}/{channel}
	
	B->>A: SSE on /notifications/{channel}
	% potentially SSE on /notifications/{DynamicId}/{channel}
	
	A->>B: POST /sessions/{DynamicId}/
	B->>A: 200 OK
	% final answer could be SSE on /notifications/{OBappId}/general
	
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
