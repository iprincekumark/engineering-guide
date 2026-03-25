# SOAP Web Services — Interview Questions & Answers

---

## 1. What is a Web Service?

A **Web Service** is a **standardized way for two applications to communicate over a network (typically HTTP) using platform-independent, language-agnostic protocols**. The client and server don't need to be written in the same language or run on the same OS — they exchange data via a defined contract.

| Property | Description |
|---|---|
| **Interoperability** | Java server can talk to a .NET client |
| **Platform-independent** | Runs over HTTP/HTTPS — works anywhere |
| **Loosely Coupled** | Client only knows the contract, not the implementation |
| **Self-describing** | Contract (WSDL/OpenAPI) describes how to call it |
| **Discoverable** | Can be registered and found via UDDI or API gateways |

### Two Main Types:

| Type | Protocol | Format | Contract | Usage |
|---|---|---|---|---|
| **SOAP** | HTTP, SMTP, JMS | XML only | WSDL (strict) | Banking, enterprise, legacy |
| **REST** | HTTP | JSON, XML, etc. | OpenAPI (optional) | Modern APIs, mobile, web |

```
Client App                     Server App
   │                               │
   │──── HTTP POST (XML/JSON) ────▶│
   │                               │  Process Request
   │◀─── HTTP Response (XML/JSON)──│
```

**Real World Analogy:** A web service is like **international mail with a standard envelope format**. No matter which country you're in, you follow the same address format (protocol), put your message inside (payload), and the postal system (HTTP) delivers it reliably to any destination.

---

## 2. What is SOAP Web Service?

A **SOAP Web Service** is a **web service that uses the SOAP protocol to exchange structured XML messages** between a client and a server. It is **contract-driven** — the entire API is described in a WSDL document, and every message must be valid XML conforming to a defined XSD schema.

| Feature | Description |
|---|---|
| **Protocol** | SOAP (Simple Object Access Protocol) over HTTP/HTTPS |
| **Message Format** | XML only — strictly structured |
| **Contract** | WSDL (Web Services Description Language) |
| **Schema Validation** | XSD validates every request and response |
| **Error Format** | SOAP Fault — standardized error structure |
| **Security** | WS-Security header for token/credential passing |
| **Transactions** | WS-AtomicTransaction support |

### When SOAP is Preferred Over REST:

| Scenario | Reason |
|---|---|
| Banking / financial transactions | Built-in ACID transaction support |
| Enterprise B2B integration | Strict contract + WS-Security |
| Legacy system integration | SOAP has been the enterprise standard for 20+ years |
| Guaranteed message delivery | WS-ReliableMessaging support |
| Formal contract required | WSDL provides machine-readable, typed contract |

```xml
<!-- Every SOAP interaction is an XML message — strict structure -->
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/">
    <soapenv:Header/>
    <soapenv:Body>
        <GetUserRequest>
            <userId>42</userId>
        </GetUserRequest>
    </soapenv:Body>
</soapenv:Envelope>
```

**Real World Analogy:** A SOAP Web Service is like **a certified legal courier service**. Every package (message) must be in a specific format (SOAP XML), come with a cover letter (WSDL contract), and be signed off (schema validation). It's not the fastest, but it's the most formal and traceable.

---

## 3. What is SOAP?

**SOAP (Simple Object Access Protocol)** is an **XML-based messaging protocol** used to exchange structured information between applications over a network. It defines a strict envelope format that every message must follow — regardless of transport (HTTP, SMTP, JMS).

| Property | Value |
|---|---|
| **Full Form** | Simple Object Access Protocol |
| **Maintained By** | W3C |
| **Message Format** | XML |
| **Transport** | HTTP, HTTPS, SMTP, JMS, TCP |
| **Encoding** | UTF-8 XML |
| **Version** | SOAP 1.1 (most common), SOAP 1.2 |

### SOAP Message Structure:

```
┌────────────────────────────────┐
│         SOAP Envelope          │  ← Root element — wraps everything
│  ┌──────────────────────────┐  │
│  │       SOAP Header        │  │  ← Optional — auth tokens, correlation IDs
│  └──────────────────────────┘  │
│  ┌──────────────────────────┐  │
│  │        SOAP Body         │  │  ← Mandatory — actual request/response data
│  │   (or SOAP Fault)        │  │
│  └──────────────────────────┘  │
└────────────────────────────────┘
```

### SOAP 1.1 vs SOAP 1.2:

| Feature | SOAP 1.1 | SOAP 1.2 |
|---|---|---|
| **Namespace** | `http://schemas.xmlsoap.org/soap/envelope/` | `http://www.w3.org/2003/05/soap-envelope` |
| **HTTP Content-Type** | `text/xml` | `application/soap+xml` |
| **HTTP Method** | POST only | POST (GET for idempotent) |
| **Fault structure** | Basic | More detailed fault codes |
| **Usage** | Most common in enterprise | Newer standard |

```xml
<!-- SOAP 1.1 — most widely used -->
POST /ws HTTP/1.1
Host: example.com
Content-Type: text/xml; charset=utf-8
SOAPAction: "getUserById"

<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/">
    <soapenv:Body>
        <GetUserRequest>
            <userId>42</userId>
        </GetUserRequest>
    </soapenv:Body>
</soapenv:Envelope>
```

**Real World Analogy:** SOAP is like **a standardized government form**. You can't just send any letter — you must fill out the prescribed form (XML envelope), include the required sections (header, body), and submit through the designated channel (HTTP POST). It's bureaucratic but unambiguous.

---

## 4. What is SOAP Envelope?

The **SOAP Envelope** is the **root XML element that wraps every SOAP message**. It is the mandatory outer container that identifies the document as a SOAP message and declares the namespaces used inside it.

| Component | Mandatory | Purpose |
|---|---|---|
| `<Envelope>` | ✅ Yes | Root element — identifies the SOAP message |
| `<Header>` | ❌ Optional | Metadata — auth, routing, transaction info |
| `<Body>` | ✅ Yes | Actual payload — request or response data |
| `<Fault>` | ❌ Only on error | Error details — inside Body on failure |

### Envelope Structure:

```xml
<!-- SOAP 1.1 Envelope — the required outer wrapper -->
<soapenv:Envelope
    xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/"
    xmlns:usr="http://example.com/user">

    <!-- Header: optional — metadata, security tokens, routing info -->
    <soapenv:Header>
        <usr:RequestId>REQ-20240101-001</usr:RequestId>
    </soapenv:Header>

    <!-- Body: mandatory — the actual message payload -->
    <soapenv:Body>
        <usr:GetUserRequest>
            <usr:userId>42</usr:userId>
        </usr:GetUserRequest>
    </soapenv:Body>

</soapenv:Envelope>
```

### Key Rules:

```xml
<!-- ❌ WRONG — Envelope without namespace declaration -->
<Envelope>
    <Body>...</Body>
</Envelope>

<!-- ✅ CORRECT — Namespace is mandatory -->
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/">
    <soapenv:Body>...</soapenv:Body>
</soapenv:Envelope>

<!-- ✅ CORRECT — Header must come BEFORE Body if present -->
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/">
    <soapenv:Header>...</soapenv:Header>   <!-- Header first -->
    <soapenv:Body>...</soapenv:Body>       <!-- Body second -->
</soapenv:Envelope>
```

**Real World Analogy:** The SOAP Envelope is like a **physical envelope for a letter**. It has a defined outside structure (recipient, sender, stamps on the cover = Header), and the actual letter inside (Body). Without the envelope, the postal service (transport) cannot process it.

---

## 5. SOAP Header and Body?

### SOAP Header:

The **SOAP Header** is an **optional element inside the Envelope** used to pass **metadata** — information that is not part of the core business message but is needed for processing: authentication tokens, request IDs, routing instructions, transaction context.

```xml
<soapenv:Header>

    <!-- Authentication token — consumed by a security handler -->
    <wsse:Security xmlns:wsse="http://docs.oasis-open.org/wss/...">
        <wsse:UsernameToken>
            <wsse:Username>john.doe</wsse:Username>
            <wsse:Password>secret123</wsse:Password>
        </wsse:UsernameToken>
    </wsse:Security>

    <!-- Correlation ID — for distributed tracing -->
    <tns:CorrelationId>REQ-2024-XYZ-001</tns:CorrelationId>

    <!-- mustUnderstand="1" — receiver MUST process this header or fault -->
    <tns:TransactionId soapenv:mustUnderstand="1">TXN-9988</tns:TransactionId>

</soapenv:Header>
```

### SOAP Body:

The **SOAP Body** is the **mandatory element** that contains the actual **business payload** — the request being made or the response being returned. Every meaningful data exchange lives here.

```xml
<!-- REQUEST BODY — calling GetUser operation -->
<soapenv:Body>
    <tns:GetUserRequest xmlns:tns="http://example.com/user">
        <tns:userId>42</tns:userId>
    </tns:GetUserRequest>
</soapenv:Body>

<!-- RESPONSE BODY — server's response -->
<soapenv:Body>
    <tns:GetUserResponse xmlns:tns="http://example.com/user">
        <tns:user>
            <tns:id>42</tns:id>
            <tns:name>John Doe</tns:name>
            <tns:email>john.doe@example.com</tns:email>
        </tns:user>
    </tns:GetUserResponse>
</soapenv:Body>

<!-- ERROR BODY — fault when something goes wrong -->
<soapenv:Body>
    <soapenv:Fault>
        <faultcode>soapenv:Client</faultcode>
        <faultstring>User with ID 42 not found</faultstring>
    </soapenv:Fault>
</soapenv:Body>
```

### Header vs Body — At a Glance:

| | Header | Body |
|---|---|---|
| **Mandatory** | ❌ No | ✅ Yes |
| **Purpose** | Metadata / infrastructure | Business payload |
| **Examples** | Auth token, trace ID, routing | GetUserRequest, CreateOrderResponse |
| **`mustUnderstand`** | Supported | N/A |
| **Processed by** | Intermediaries + endpoint | Endpoint only |

**Real World Analogy:** In a physical parcel: the **Header** is the label on the outside (sender, recipient, courier instructions, fragile stickers) — the logistics layer. The **Body** is the actual item inside the parcel (what was ordered). You need both, but the carrier handles the label; the recipient cares about the contents.

---

## 6. Example SOAP Request/Response?

A complete, real-world SOAP request/response cycle for a `GetUser` operation in a Spring WS application.

### SOAP Request (Client → Server):

```xml
POST /ws HTTP/1.1
Host: localhost:8080
Content-Type: text/xml; charset=utf-8
SOAPAction: ""

<soapenv:Envelope
    xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/"
    xmlns:tns="http://example.com/user">

    <soapenv:Header/>

    <soapenv:Body>
        <tns:GetUserRequest>
            <tns:userId>42</tns:userId>
        </tns:GetUserRequest>
    </soapenv:Body>

</soapenv:Envelope>
```

### SOAP Response (Server → Client) — Success:

```xml
HTTP/1.1 200 OK
Content-Type: text/xml; charset=utf-8

<soapenv:Envelope
    xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/"
    xmlns:tns="http://example.com/user">

    <soapenv:Header/>

    <soapenv:Body>
        <tns:GetUserResponse>
            <tns:user>
                <tns:id>42</tns:id>
                <tns:name>John Doe</tns:name>
                <tns:email>john.doe@example.com</tns:email>
                <tns:department>Engineering</tns:department>
                <tns:active>true</tns:active>
            </tns:user>
        </tns:GetUserResponse>
    </soapenv:Body>

</soapenv:Envelope>
```

### SOAP Response — Fault (Error):

```xml
HTTP/1.1 500 Internal Server Error
Content-Type: text/xml; charset=utf-8

<soapenv:Envelope
    xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/">

    <soapenv:Body>
        <soapenv:Fault>
            <faultcode>soapenv:Client</faultcode>
            <faultstring>User not found with ID: 42</faultstring>
            <detail>
                <errorCode>USER_NOT_FOUND</errorCode>
                <errorMessage>No user exists with the provided ID</errorMessage>
            </detail>
        </soapenv:Fault>
    </soapenv:Body>

</soapenv:Envelope>
```

### Spring WS Java Endpoint (Processes the Above):

```java
@Endpoint
public class UserEndpoint {

    private static final String NAMESPACE = "http://example.com/user";

    @Autowired
    private UserService userService;

    // ==================== HANDLES GetUserRequest ====================
    @PayloadRoot(namespace = NAMESPACE, localPart = "GetUserRequest")
    @ResponsePayload
    public GetUserResponse getUser(@RequestPayload GetUserRequest request) {

        User user = userService.findById(request.getUserId());   // Business logic

        GetUserResponse response = new GetUserResponse();
        response.setUser(user);
        return response;                                          // Spring WS serializes to XML
    }
}
```

**Real World Analogy:** A SOAP request/response is like **calling a company's customer service hotline with a scripted phone tree**. You dial (HTTP POST), follow the exact prompts (XML schema), give your account number (userId in Body), and receive a structured response read off a script (GetUserResponse). Wrong input → scripted error message (SOAP Fault).

---

## 7. SOAP Header Usage?

The SOAP Header is used to pass **metadata that supports the processing of the message** — without polluting the business payload in the Body. It enables cross-cutting concerns in SOAP services: authentication, tracing, routing, and transactions.

| Header Use Case | What It Contains |
|---|---|
| **Authentication** | Username/password, API key, OAuth token, WS-Security token |
| **Request Tracing** | Correlation ID, request timestamp, client app ID |
| **Routing** | Which backend service or version to route to |
| **Transaction Context** | Transaction ID for distributed ACID operations |
| **Idempotency** | Message ID to prevent duplicate processing |
| **Locale / Language** | `Accept-Language` for internationalized responses |

### Custom Header in Spring WS — Reading Header in Endpoint:

```java
@Endpoint
public class UserEndpoint {

    private static final String NAMESPACE = "http://example.com/user";

    // ==================== READ CUSTOM SOAP HEADER ====================
    @PayloadRoot(namespace = NAMESPACE, localPart = "GetUserRequest")
    @ResponsePayload
    public GetUserResponse getUser(
            @RequestPayload GetUserRequest request,
            MessageContext messageContext) throws Exception {

        // Access the raw SOAP message to read headers
        SaajSoapMessage soapMessage =
            (SaajSoapMessage) messageContext.getRequest();

        SOAPHeader header = soapMessage.getSaajMessage().getSOAPHeader();

        // Read a custom CorrelationId header
        Iterator<?> iter = header.examineAllHeaderElements();
        while (iter.hasNext()) {
            SOAPHeaderElement element = (SOAPHeaderElement) iter.next();
            if ("CorrelationId".equals(element.getLocalName())) {
                String correlationId = element.getTextContent();
                MDC.put("correlationId", correlationId);    // Add to logging context
            }
        }

        return userService.findById(request.getUserId());
    }
}
```

### mustUnderstand Attribute:

```xml
<!-- mustUnderstand="1" — the server MUST process this header.
     If it doesn't understand it, it MUST return a SOAP Fault. -->
<soapenv:Header>
    <tns:TransactionId soapenv:mustUnderstand="1">TXN-7890</tns:TransactionId>
</soapenv:Header>

<!-- mustUnderstand="0" or absent — server MAY ignore this header safely -->
<soapenv:Header>
    <tns:ClientAppVersion>2.3.1</tns:ClientAppVersion>
</soapenv:Header>
```

**Real World Analogy:** SOAP Headers are like **sticky notes on the outside of a sealed envelope**. They carry instructions for the courier or the receptionist (route to floor 5, handle with care, requires signature) without opening the letter itself (Body). The business content stays untouched; the infrastructure reads the notes.

---

## 8. SOAP Header with Authentication?

**WS-Security** is the standard OASIS specification for adding authentication credentials to SOAP Headers — placing username/password or security tokens in the Header so the server can authenticate the caller before processing the Body.

### Option 1 — UsernameToken in Header (Basic Auth):

```xml
<!-- Client sends credentials in SOAP Header -->
<soapenv:Envelope
    xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/"
    xmlns:wsse="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-wssecurity-secext-1.0.xsd">

    <soapenv:Header>
        <wsse:Security soapenv:mustUnderstand="1">
            <wsse:UsernameToken>
                <wsse:Username>john.doe</wsse:Username>
                <!-- PasswordDigest or PasswordText -->
                <wsse:Password Type="...#PasswordText">secret123</wsse:Password>
            </wsse:UsernameToken>
        </wsse:Security>
    </soapenv:Header>

    <soapenv:Body>
        <tns:GetUserRequest>
            <tns:userId>42</tns:userId>
        </tns:GetUserRequest>
    </soapenv:Body>

</soapenv:Envelope>
```

### Option 2 — Spring WS Security Interceptor (Server-Side):

```java
// ==================== SPRING WS SECURITY CONFIG ====================
@Configuration
@EnableWs
public class WsSecurityConfig extends WsConfigurerAdapter {

    @Bean
    public Wss4jSecurityInterceptor securityInterceptor() {
        Wss4jSecurityInterceptor interceptor = new Wss4jSecurityInterceptor();

        // Validate UsernameToken in every incoming request
        interceptor.setValidationActions("UsernameToken");

        // Provide a UserDetailsService to validate credentials
        interceptor.setValidationCallbackHandler(
            new SimplePasswordValidationCallbackHandler() {{
                setUsersMap(Map.of("john.doe", "secret123"));
            }}
        );

        return interceptor;
    }

    @Override
    public void addInterceptors(List<EndpointInterceptor> interceptors) {
        interceptors.add(securityInterceptor());   // Applied globally to all endpoints
    }
}
```

### Option 3 — Custom Header Auth Interceptor:

```java
// ==================== CUSTOM HEADER INTERCEPTOR ====================
@Component
public class AuthHeaderInterceptor implements EndpointInterceptor {

    @Override
    public boolean handleRequest(
            MessageContext messageContext,
            Object endpoint) throws Exception {

        SaajSoapMessage request = (SaajSoapMessage) messageContext.getRequest();
        SOAPHeader header = request.getSaajMessage().getSOAPHeader();

        // Read Authorization header element
        NodeList authNodes = header.getElementsByTagNameNS(
            "http://example.com/security", "ApiKey");

        if (authNodes.getLength() == 0) {
            throw new SoapFaultException("Missing Authorization header");
        }

        String apiKey = authNodes.item(0).getTextContent();
        if (!apiKeyService.isValid(apiKey)) {
            throw new SoapFaultException("Invalid API Key");
        }

        return true;  // ✅ Proceed to endpoint
    }

    @Override
    public boolean handleResponse(MessageContext ctx, Object endpoint) { return true; }

    @Override
    public boolean handleFault(MessageContext ctx, Object endpoint) { return true; }

    @Override
    public void afterCompletion(MessageContext ctx, Object endpoint, Exception ex) {}
}
```

**Real World Analogy:** SOAP Header authentication is like a **building security badge check**. Every visitor (SOAP request) must present a badge (UsernameToken/ApiKey) at the front desk (interceptor) before being allowed into the office floors (endpoint processing). The badge check is separate from what the visitor is there to do (Body payload).

---

## 9. What is WSDL?

**WSDL (Web Services Description Language)** is an **XML document that describes a SOAP web service completely** — what operations it provides, what XML messages each operation expects as input, what it returns, and where the service is hosted. It is the **contract** between client and server.

| Property | Description |
|---|---|
| **Full Form** | Web Services Description Language |
| **Format** | XML |
| **Purpose** | Machine-readable contract for a SOAP service |
| **Version** | WSDL 1.1 (most common), WSDL 2.0 |
| **Maintained By** | W3C |
| **Auto-generated?** | ✅ Yes — Spring WS generates it from XSD |

### Why WSDL Matters:

```
Without WSDL:                         With WSDL:
──────────────────────────────        ────────────────────────────────
- Client must guess the XML format    - Client reads WSDL → knows exact XML
- No schema validation possible       - Schema enforced on every message
- Integration requires phone calls    - Tools auto-generate client code
- Fragile, undocumented contracts     - Formal, versioned, machine-readable
```

### What WSDL Enables:

```
WSDL URL: http://localhost:8080/ws/users.wsdl
       │
       ├──▶  wsimport (JDK tool) → auto-generates Java client stubs
       ├──▶  SoapUI → auto-generates test cases
       ├──▶  .NET / Python / PHP → auto-generates clients
       └──▶  API Gateways → auto-import and publish service
```

**Real World Analogy:** WSDL is like a **restaurant menu with recipes attached**. The menu tells customers what dishes are available (operations), what ingredients you need to order them (input message schema), and what you'll receive on your plate (output message schema). The kitchen (server) and diner (client) both follow the same menu — no surprises.

---

## 10. Parts of WSDL?

A WSDL document has **6 main sections** — each describing a different aspect of the service.

| Section | Tag | Purpose |
|---|---|---|
| **Types** | `<types>` | XSD schemas defining all request/response XML structures |
| **Message** | `<message>` | Named input/output message definitions (uses Types) |
| **PortType** | `<portType>` | Abstract operations — names + input/output messages |
| **Binding** | `<binding>` | Concrete protocol — SOAP 1.1/1.2, document/rpc style |
| **Service** | `<service>` | Physical location — the actual URL endpoint |
| **Port** | `<port>` | A single endpoint inside Service — binding + address |

### Annotated WSDL Example:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<wsdl:definitions
    xmlns:wsdl="http://schemas.xmlsoap.org/wsdl/"
    xmlns:soap="http://schemas.xmlsoap.org/wsdl/soap/"
    xmlns:tns="http://example.com/user"
    xmlns:xsd="http://www.w3.org/2001/XMLSchema"
    targetNamespace="http://example.com/user"
    name="UserService">

    <!-- ===== TYPES: XSD schema for messages ===== -->
    <wsdl:types>
        <xsd:schema targetNamespace="http://example.com/user">
            <xsd:element name="GetUserRequest">
                <xsd:complexType>
                    <xsd:sequence>
                        <xsd:element name="userId" type="xsd:int"/>
                    </xsd:sequence>
                </xsd:complexType>
            </xsd:element>
            <xsd:element name="GetUserResponse">
                <xsd:complexType>
                    <xsd:sequence>
                        <xsd:element name="name"  type="xsd:string"/>
                        <xsd:element name="email" type="xsd:string"/>
                    </xsd:sequence>
                </xsd:complexType>
            </xsd:element>
        </xsd:schema>
    </wsdl:types>

    <!-- ===== MESSAGE: Named message definitions ===== -->
    <wsdl:message name="GetUserRequestMsg">
        <wsdl:part name="parameters" element="tns:GetUserRequest"/>
    </wsdl:message>
    <wsdl:message name="GetUserResponseMsg">
        <wsdl:part name="parameters" element="tns:GetUserResponse"/>
    </wsdl:message>

    <!-- ===== PORT TYPE: Abstract operations ===== -->
    <wsdl:portType name="UserPortType">
        <wsdl:operation name="GetUser">
            <wsdl:input  message="tns:GetUserRequestMsg"/>
            <wsdl:output message="tns:GetUserResponseMsg"/>
        </wsdl:operation>
    </wsdl:portType>

    <!-- ===== BINDING: Concrete protocol details ===== -->
    <wsdl:binding name="UserBinding" type="tns:UserPortType">
        <soap:binding style="document"
                      transport="http://schemas.xmlsoap.org/soap/http"/>
        <wsdl:operation name="GetUser">
            <soap:operation soapAction=""/>
            <wsdl:input>  <soap:body use="literal"/> </wsdl:input>
            <wsdl:output> <soap:body use="literal"/> </wsdl:output>
        </wsdl:operation>
    </wsdl:binding>

    <!-- ===== SERVICE: Physical location ===== -->
    <wsdl:service name="UserService">
        <wsdl:port name="UserPort" binding="tns:UserBinding">
            <soap:address location="http://localhost:8080/ws"/>
        </wsdl:port>
    </wsdl:service>

</wsdl:definitions>
```

**Real World Analogy:** Think of a WSDL like a **job contract document**. Types = skill requirements (what inputs/outputs look like). PortType = job responsibilities (operations). Binding = working terms (protocol, style). Service = office address (where to show up). Message = the specific deliverables for each task.

---

## 11. Contract First Approach?

**Contract First** (also called **WSDL First**) means **you define the XSD schema first, then generate Java code from it** — the schema is the source of truth. This is the **recommended approach for Spring WS**.

| Approach | Description | Verdict |
|---|---|---|
| **Contract First** | Write XSD → generate Java classes (JAXB) | ✅ Recommended — clean, schema-validated |
| **Code First** | Write Java → generate WSDL/XSD automatically | ⚠️ Easier start but fragile contracts |

### Why Contract First is Better:

```
CONTRACT FIRST FLOW:
─────────────────────────────────────────────────────────────
 1. Define XSD (users.xsd)          ← Source of truth
 2. JAXB generates Java POJOs       ← from XSD
 3. Spring WS generates WSDL        ← from XSD
 4. Endpoint uses generated classes ← type-safe, validated
 5. Client reads WSDL               ← auto-generates stubs

 Result: Every message is schema-validated automatically.
         Change the XSD → everything regenerates consistently.

CODE FIRST FLOW:
─────────────────────────────────────────────────────────────
 1. Write Java classes               ← Source of truth
 2. Tool generates WSDL from Java    ← often imprecise
 3. Schema validation is weak        ← Java types don't map cleanly to XSD
 4. Client couples to implementation ← breaking changes are invisible
```

### Contract First Project Structure:

```
src/
  main/
    resources/
      wsdl/
        users.xsd          ← ✅ START HERE — define your contract
    java/
      com/example/
        endpoint/
          UserEndpoint.java  ← Uses JAXB-generated classes
        config/
          WsConfig.java      ← Registers WSDL + MessageDispatcherServlet
target/
  generated-sources/
    jaxb/
      com/example/           ← ✅ Auto-generated from XSD — never edit manually
        GetUserRequest.java
        GetUserResponse.java
        User.java
```

> **Best Practice:** Always use **Contract First** with Spring WS. Define your `.xsd` file first, configure the JAXB Maven plugin to generate classes, and let Spring WS auto-generate the WSDL. This ensures schema validation on every request with zero extra code.

**Real World Analogy:** Contract First is like **an architect drawing blueprints before construction**. Everyone — builders (server), inspectors (validators), and clients (API consumers) — works from the same blueprint (XSD). Code First is like building first and drawing the blueprint afterward — the blueprint is often inaccurate.

---

## 12. What is XSD?

**XSD (XML Schema Definition)** is an **XML document that defines the structure, data types, and constraints for XML messages**. In SOAP web services, XSD defines exactly what a valid request and response must look like — and Spring WS uses it to validate every incoming and outgoing message automatically.

| Property | Description |
|---|---|
| **Full Form** | XML Schema Definition |
| **File Extension** | `.xsd` |
| **Purpose** | Defines valid structure and types for XML documents |
| **Used By** | JAXB (generates Java classes), WSDL (references types), Spring WS (validation) |
| **Standard** | W3C XML Schema |

### XSD Data Types:

| XSD Type | Java Type | Example |
|---|---|---|
| `xsd:string` | `String` | `"John Doe"` |
| `xsd:int` | `int` / `Integer` | `42` |
| `xsd:boolean` | `boolean` / `Boolean` | `true` |
| `xsd:decimal` | `BigDecimal` | `99.99` |
| `xsd:date` | `XMLGregorianCalendar` | `2024-01-15` |
| `xsd:dateTime` | `XMLGregorianCalendar` | `2024-01-15T10:30:00` |

### XSD Constraints:

```xml
<!-- XSD supports built-in validation constraints -->

<!-- String with min/max length -->
<xsd:element name="username">
    <xsd:simpleType>
        <xsd:restriction base="xsd:string">
            <xsd:minLength value="3"/>
            <xsd:maxLength value="50"/>
        </xsd:restriction>
    </xsd:simpleType>
</xsd:element>

<!-- Number with range -->
<xsd:element name="age">
    <xsd:simpleType>
        <xsd:restriction base="xsd:int">
            <xsd:minInclusive value="0"/>
            <xsd:maxInclusive value="150"/>
        </xsd:restriction>
    </xsd:simpleType>
</xsd:element>

<!-- Enum / fixed values -->
<xsd:element name="status">
    <xsd:simpleType>
        <xsd:restriction base="xsd:string">
            <xsd:enumeration value="ACTIVE"/>
            <xsd:enumeration value="INACTIVE"/>
            <xsd:enumeration value="SUSPENDED"/>
        </xsd:restriction>
    </xsd:simpleType>
</xsd:element>

<!-- Email pattern -->
<xsd:element name="email">
    <xsd:simpleType>
        <xsd:restriction base="xsd:string">
            <xsd:pattern value="[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}"/>
        </xsd:restriction>
    </xsd:simpleType>
</xsd:element>
```

**Real World Analogy:** XSD is like a **government form template**. It specifies exactly which fields exist, which are mandatory, what type of value each accepts (numbers only, date format, max characters), and what values are valid (e.g., country code must be one of a defined list). Fill it wrong — it gets rejected at the counter.

---

## 13. Example XSD?

A complete, production-realistic XSD for a `UserService` SOAP web service — covering `GetUser`, `CreateUser`, and `DeleteUser` operations.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!-- users.xsd — place in src/main/resources/wsdl/ -->
<xsd:schema
    xmlns:xsd="http://www.w3.org/2001/XMLSchema"
    xmlns:tns="http://example.com/user"
    targetNamespace="http://example.com/user"
    elementFormDefault="qualified">

    <!-- ==================== SHARED TYPE: User ==================== -->
    <xsd:complexType name="User">
        <xsd:sequence>
            <xsd:element name="id"         type="xsd:int"/>
            <xsd:element name="name"       type="xsd:string"/>
            <xsd:element name="email"      type="xsd:string"/>
            <xsd:element name="department" type="xsd:string"  minOccurs="0"/>
            <xsd:element name="active"     type="xsd:boolean" default="true"/>
        </xsd:sequence>
    </xsd:complexType>

    <!-- ==================== GET USER ==================== -->
    <xsd:element name="GetUserRequest">
        <xsd:complexType>
            <xsd:sequence>
                <xsd:element name="userId" type="xsd:int"/>
            </xsd:sequence>
        </xsd:complexType>
    </xsd:element>

    <xsd:element name="GetUserResponse">
        <xsd:complexType>
            <xsd:sequence>
                <xsd:element name="user" type="tns:User"/>
            </xsd:sequence>
        </xsd:complexType>
    </xsd:element>

    <!-- ==================== CREATE USER ==================== -->
    <xsd:element name="CreateUserRequest">
        <xsd:complexType>
            <xsd:sequence>
                <xsd:element name="name"       type="xsd:string"/>
                <xsd:element name="email"      type="xsd:string"/>
                <xsd:element name="department" type="xsd:string" minOccurs="0"/>
            </xsd:sequence>
        </xsd:complexType>
    </xsd:element>

    <xsd:element name="CreateUserResponse">
        <xsd:complexType>
            <xsd:sequence>
                <xsd:element name="user"    type="tns:User"/>
                <xsd:element name="message" type="xsd:string"/>
            </xsd:sequence>
        </xsd:complexType>
    </xsd:element>

    <!-- ==================== DELETE USER ==================== -->
    <xsd:element name="DeleteUserRequest">
        <xsd:complexType>
            <xsd:sequence>
                <xsd:element name="userId" type="xsd:int"/>
            </xsd:sequence>
        </xsd:complexType>
    </xsd:element>

    <xsd:element name="DeleteUserResponse">
        <xsd:complexType>
            <xsd:sequence>
                <xsd:element name="success" type="xsd:boolean"/>
                <xsd:element name="message" type="xsd:string"/>
            </xsd:sequence>
        </xsd:complexType>
    </xsd:element>

    <!-- ==================== SOAP FAULT DETAIL ==================== -->
    <xsd:element name="ServiceFaultDetail">
        <xsd:complexType>
            <xsd:sequence>
                <xsd:element name="errorCode"    type="xsd:string"/>
                <xsd:element name="errorMessage" type="xsd:string"/>
                <xsd:element name="timestamp"    type="xsd:dateTime"/>
            </xsd:sequence>
        </xsd:complexType>
    </xsd:element>

</xsd:schema>
```

### JAXB Will Generate These Java Classes from the XSD:

```java
// Auto-generated — never edit manually
// target/generated-sources/jaxb/com/example/user/

GetUserRequest.java       // getters/setters for userId
GetUserResponse.java      // getters/setters for user (User type)
CreateUserRequest.java    // getters/setters for name, email, department
CreateUserResponse.java   // getters/setters for user, message
DeleteUserRequest.java    // getters/setters for userId
DeleteUserResponse.java   // getters/setters for success, message
User.java                 // getters/setters for id, name, email, department, active
```

**Real World Analogy:** An XSD file is like a **database schema script**. Just as `CREATE TABLE users (id INT, name VARCHAR(100), email VARCHAR(200))` defines the structure for all user records, an XSD defines the exact structure for all XML messages. Everything inserted (sent) is validated against the schema automatically.

---

## 14. What is JAXB?

**JAXB (Java Architecture for XML Binding)** is a **Java framework that converts Java objects to XML (marshalling) and XML back to Java objects (unmarshalling)**. In Spring WS, JAXB is used to auto-generate Java classes from XSD schemas, enabling type-safe access to SOAP message data.

| Term | Meaning |
|---|---|
| **Marshalling** | Java Object → XML (used when sending SOAP response) |
| **Unmarshalling** | XML → Java Object (used when receiving SOAP request) |
| **Schema Binding** | Generates annotated Java POJOs from XSD automatically |
| **`@XmlRootElement`** | Marks a class as the root of an XML document |
| **`@XmlElement`** | Maps a Java field to an XML element |

### JAXB Annotations (Auto-generated by Plugin):

```java
// ✅ Auto-generated by JAXB from XSD — this is what you get in target/generated-sources
@XmlAccessorType(XmlAccessType.FIELD)
@XmlType(name = "User", propOrder = { "id", "name", "email", "department", "active" })
public class User {

    @XmlElement(required = true)
    protected int id;

    @XmlElement(required = true)
    protected String name;

    @XmlElement(required = true)
    protected String email;

    protected String department;     // minOccurs="0" in XSD → not required

    @XmlElement(defaultValue = "true")
    protected boolean active;

    // Getters and setters auto-generated...
}

@XmlRootElement(name = "GetUserRequest", namespace = "http://example.com/user")
@XmlAccessorType(XmlAccessType.FIELD)
public class GetUserRequest {

    @XmlElement(required = true)
    protected int userId;

    // Getter/setter...
}
```

### Manual JAXB Usage (Marshalling / Unmarshalling):

```java
// ==================== MARSHAL — Java to XML ====================
JAXBContext context = JAXBContext.newInstance(GetUserResponse.class);
Marshaller marshaller = context.createMarshaller();
marshaller.setProperty(Marshaller.JAXB_FORMATTED_OUTPUT, true);

GetUserResponse response = new GetUserResponse();
response.setUser(user);

marshaller.marshal(response, System.out);
// Output: <GetUserResponse><user><id>42</id>...

// ==================== UNMARSHAL — XML to Java ====================
JAXBContext context = JAXBContext.newInstance(GetUserRequest.class);
Unmarshaller unmarshaller = context.createUnmarshaller();

GetUserRequest request = (GetUserRequest)
    unmarshaller.unmarshal(new StringReader(xmlString));
System.out.println(request.getUserId()); // 42
```

**Real World Analogy:** JAXB is like a **universal translator at the UN**. When a Java delegate (object) wants to speak (send a message), the translator converts their spoken words to written text (marshalling: Java → XML). When written text arrives (XML received), the translator reads it aloud for the Java delegate (unmarshalling: XML → Java). Both sides work natively in their preferred format.

---

## 15. Configure JAXB Plugin?

The **JAXB Maven Plugin** (`jaxb2-maven-plugin`) is configured in `pom.xml` to **automatically generate Java classes from your XSD at build time** — triggered during the Maven `generate-sources` phase.

### Maven Dependency + Plugin Setup:

```xml
<!-- pom.xml -->

<!-- ==================== DEPENDENCIES ==================== -->
<dependencies>

    <!-- Spring WS core -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web-services</artifactId>
    </dependency>

    <!-- WSDL4J — required for WSDL generation in Spring WS -->
    <dependency>
        <groupId>wsdl4j</groupId>
        <artifactId>wsdl4j</artifactId>
    </dependency>

    <!-- JAXB API + Runtime (needed for Java 11+) -->
    <dependency>
        <groupId>jakarta.xml.bind</groupId>
        <artifactId>jakarta.xml.bind-api</artifactId>
    </dependency>
    <dependency>
        <groupId>com.sun.xml.bind</groupId>
        <artifactId>jaxb-impl</artifactId>
        <version>4.0.4</version>
    </dependency>

</dependencies>

<!-- ==================== JAXB PLUGIN ==================== -->
<build>
    <plugins>
        <plugin>
            <groupId>org.codehaus.mojo</groupId>
            <artifactId>jaxb2-maven-plugin</artifactId>
            <version>3.1.0</version>
            <executions>
                <execution>
                    <id>xjc-generate</id>
                    <goals>
                        <goal>xjc</goal>   <!-- XJC = XML to Java Compiler -->
                    </goals>
                </execution>
            </executions>
            <configuration>
                <!-- Source XSD location -->
                <sources>
                    <source>src/main/resources/wsdl/users.xsd</source>
                </sources>
                <!-- Output package for generated classes -->
                <packageName>com.example.user.generated</packageName>
                <!-- Output directory (auto-added to compile classpath) -->
                <outputDirectory>${project.build.directory}/generated-sources/jaxb</outputDirectory>
                <clearOutputDir>false</clearOutputDir>
            </configuration>
        </plugin>
    </plugins>
</build>
```

### Build and Verify:

```bash
# Run JAXB code generation
mvn generate-sources

# Verify generated classes in:
# target/generated-sources/jaxb/com/example/user/generated/
#   ├── GetUserRequest.java
#   ├── GetUserResponse.java
#   ├── CreateUserRequest.java
#   ├── CreateUserResponse.java
#   ├── User.java
#   └── ObjectFactory.java   ← JAXB factory — do not delete
```

### IntelliJ IDEA — Mark as Source Root:

```
target/generated-sources/jaxb  →  Right-click → "Mark Directory as" → "Generated Sources Root"
```

> **Best Practice:** Never manually edit files inside `target/generated-sources/`. They are **regenerated on every build**. If you need to customize generated classes, use JAXB bindings (`.xjb` files) or wrap them in separate service/mapper classes.

**Real World Analogy:** The JAXB plugin is like a **CAD software import tool**. Your architect provides a blueprint (XSD). The tool reads the blueprint and automatically generates 3D model components (Java classes) that builders (developers) can directly use in construction (code). Change the blueprint → reimport → updated components. Nobody draws the 3D model by hand.

---

## 16. What is an Endpoint?

An **Endpoint** in Spring WS is a **Java class annotated with `@Endpoint`** that **handles incoming SOAP requests**. It is the equivalent of a `@RestController` in Spring MVC — but for SOAP messages. Each method inside handles a specific SOAP operation identified by its XML local name and namespace.

| Spring MVC | Spring WS Equivalent |
|---|---|
| `@RestController` | `@Endpoint` |
| `@RequestMapping("/path")` | `@PayloadRoot(namespace, localPart)` |
| `@RequestBody` | `@RequestPayload` |
| `@ResponseBody` | `@ResponsePayload` |
| Returns Java object (JSON) | Returns Java object (marshalled to XML by JAXB) |

### How Spring WS Routing Works:

```
Incoming SOAP Request
        │
        ▼
MessageDispatcherServlet  (Spring WS front controller)
        │
        ▼
PayloadRootAnnotationMethodEndpointMapping
        │   Reads the root XML element's namespace + localPart
        ▼
Finds @Endpoint method with matching @PayloadRoot
        │
        ▼
@RequestPayload → JAXB unmarshal XML → Java object
        │
        ▼
Endpoint method executes business logic
        │
        ▼
@ResponsePayload → JAXB marshal Java object → XML response
```

**Real World Analogy:** An Endpoint is like a **department in a company**. The receptionist (MessageDispatcherServlet) receives every visitor (SOAP request) and routes them to the correct department (Endpoint method) based on the type of request (PayloadRoot). Each department handles its specific requests and sends back a formal response.

---

## 17. Example Endpoint in Spring WS?

A complete, production-ready Spring WS `@Endpoint` handling three SOAP operations: `GetUser`, `CreateUser`, and `DeleteUser`.

```java
// ==================== SPRING WS ENDPOINT ====================
@Endpoint
public class UserEndpoint {

    private static final String NAMESPACE_URI = "http://example.com/user";

    @Autowired
    private UserService userService;

    // ==================== GET USER ====================
    @PayloadRoot(namespace = NAMESPACE_URI, localPart = "GetUserRequest")
    @ResponsePayload
    public GetUserResponse getUser(@RequestPayload GetUserRequest request) {

        // JAXB has already unmarshalled the XML → GetUserRequest Java object
        int userId = request.getUserId();

        com.example.model.User user = userService.findById(userId);

        if (user == null) {
            throw new UserNotFoundException("User not found: " + userId);
        }

        // Build and return response — JAXB marshals this to XML automatically
        GetUserResponse response = new GetUserResponse();

        com.example.user.generated.User xmlUser = new com.example.user.generated.User();
        xmlUser.setId(user.getId());
        xmlUser.setName(user.getName());
        xmlUser.setEmail(user.getEmail());
        xmlUser.setDepartment(user.getDepartment());
        xmlUser.setActive(user.isActive());

        response.setUser(xmlUser);
        return response;
    }

    // ==================== CREATE USER ====================
    @PayloadRoot(namespace = NAMESPACE_URI, localPart = "CreateUserRequest")
    @ResponsePayload
    public CreateUserResponse createUser(@RequestPayload CreateUserRequest request) {

        com.example.model.User newUser = new com.example.model.User();
        newUser.setName(request.getName());
        newUser.setEmail(request.getEmail());
        newUser.setDepartment(request.getDepartment());

        com.example.model.User saved = userService.save(newUser);

        CreateUserResponse response = new CreateUserResponse();
        // ... populate response
        response.setMessage("User created successfully with ID: " + saved.getId());
        return response;
    }

    // ==================== DELETE USER ====================
    @PayloadRoot(namespace = NAMESPACE_URI, localPart = "DeleteUserRequest")
    @ResponsePayload
    public DeleteUserResponse deleteUser(@RequestPayload DeleteUserRequest request) {

        boolean deleted = userService.deleteById(request.getUserId());

        DeleteUserResponse response = new DeleteUserResponse();
        response.setSuccess(deleted);
        response.setMessage(deleted ? "User deleted" : "User not found");
        return response;
    }
}
```

**Real World Analogy:** The `@Endpoint` is like a **bank teller window**. The `MessageDispatcherServlet` is the queue manager who directs customers (requests) to the right teller (method). `@PayloadRoot` is the teller's sign ("Deposits Only", "Withdrawals Only"). `@RequestPayload` is the form the customer hands in. `@ResponsePayload` is the stamped receipt the teller hands back.

---

## 18. What is MessageDispatcherServlet?

**`MessageDispatcherServlet`** is the **front controller of a Spring WS application** — equivalent to `DispatcherServlet` in Spring MVC. It receives every incoming SOAP HTTP request, delegates it to the correct `@Endpoint` method, and writes the SOAP response back.

| Property | Description |
|---|---|
| **Role** | Front controller for all SOAP requests |
| **Equivalent** | `DispatcherServlet` in Spring MVC |
| **URL Mapping** | All requests to `/ws/*` (or any configured path) |
| **Responsibility** | Route → Unmarshal → Dispatch → Marshal → Respond |

### Request Processing Flow:

```
HTTP POST /ws
        │
        ▼
MessageDispatcherServlet
        │
        ├──▶ EndpointInterceptors (pre-processing — auth, logging)
        │
        ├──▶ EndpointMapping — finds matching @Endpoint method via @PayloadRoot
        │
        ├──▶ MessageConverter/JAXB — XML → Java (unmarshal request)
        │
        ├──▶ @Endpoint method — business logic executes
        │
        ├──▶ MessageConverter/JAXB — Java → XML (marshal response)
        │
        ├──▶ EndpointInterceptors (post-processing)
        │
        ▼
HTTP Response (SOAP XML)
```

**Real World Analogy:** `MessageDispatcherServlet` is like a **hospital triage receptionist**. Every patient (SOAP request) comes through the front desk. The receptionist reads the form (PayloadRoot), checks credentials (interceptors), determines which doctor (endpoint method) to send the patient to, and ensures the doctor's notes (response) are properly formatted before handing them back to the patient.

---

## 19. Configure MessageDispatcherServlet?

`MessageDispatcherServlet` is configured as a Spring `@Bean` in a `@Configuration` class. It must be registered to intercept a URL pattern (typically `/ws/*`) and set to transform WSDL locations.

```java
// ==================== SPRING WS CONFIGURATION ====================
@EnableWs
@Configuration
public class WebServiceConfig extends WsConfigurerAdapter {

    // ==================== REGISTER MessageDispatcherServlet ====================
    @Bean
    public ServletRegistrationBean<MessageDispatcherServlet> messageDispatcherServlet(
            ApplicationContext applicationContext) {

        MessageDispatcherServlet servlet = new MessageDispatcherServlet();
        servlet.setApplicationContext(applicationContext);

        // ✅ Auto-transform WSDL address to match actual request URL
        servlet.setTransformWsdlLocations(true);

        // Register to handle all requests under /ws/
        return new ServletRegistrationBean<>(servlet, "/ws/*");
    }

    // ==================== REGISTER XSD FOR VALIDATION ====================
    @Bean(name = "users")  // "users" → exposed at /ws/users.wsdl
    public DefaultWsdl11Definition defaultWsdl11Definition(XsdSchema usersSchema) {

        DefaultWsdl11Definition definition = new DefaultWsdl11Definition();
        definition.setPortTypeName("UserPort");
        definition.setLocationUri("/ws");
        definition.setTargetNamespace("http://example.com/user");
        definition.setSchema(usersSchema);          // Points to your XSD
        return definition;
    }

    @Bean
    public XsdSchema usersSchema() {
        // ✅ Points to src/main/resources/wsdl/users.xsd
        return new SimpleXsdSchema(new ClassPathResource("wsdl/users.xsd"));
    }
}
```

### application.properties:

```properties
# Spring WS — no special properties needed for basic setup
# server.port=8080 (default)

# Optional: enable Spring WS request/response logging
logging.level.org.springframework.ws=DEBUG
logging.level.org.springframework.ws.client.MessageTracing=TRACE
logging.level.org.springframework.ws.server.MessageTracing=TRACE
```

### Verify Setup:

```
# WSDL available at:
http://localhost:8080/ws/users.wsdl

# Service endpoint accepts requests at:
http://localhost:8080/ws
```

> **Key Rule:** `setTransformWsdlLocations(true)` is critical in deployed environments. Without it, the WSDL will advertise `localhost:8080` as the service address even when deployed to `prod.example.com`.

**Real World Analogy:** Configuring `MessageDispatcherServlet` is like **setting up a post office branch**. You define the building address (`/ws/*`), install the routing systems (`defaultWsdl11Definition`), and post the service menu at the entrance (WSDL URL). Anyone who walks in gets routed to the right counter automatically.

---

## 20. Generate WSDL?

In Spring WS, **WSDL is auto-generated from your XSD** at runtime — you never write WSDL by hand. The `DefaultWsdl11Definition` bean reads the XSD and generates a complete, valid WSDL automatically.

### How Spring WS Generates WSDL:

```
users.xsd  (you write this)
     │
     ▼
DefaultWsdl11Definition bean
     │  reads XSD elements that end in "Request" and "Response"
     ├──▶ GetUserRequest   → creates GetUser operation (input message)
     ├──▶ GetUserResponse  → creates GetUser operation (output message)
     ├──▶ CreateUserRequest/Response → creates CreateUser operation
     └──▶ DeleteUserRequest/Response → creates DeleteUser operation
     │
     ▼
Auto-generated WSDL at: http://localhost:8080/ws/users.wsdl
```

### WSDL Generation Convention — XSD Naming Rule:

```
XSD element name ending in "Request"  →  WSDL operation input
XSD element name ending in "Response" →  WSDL operation output

Example:
  GetUserRequest  + GetUserResponse   →  operation: GetUser
  CreateUserRequest + CreateUserResponse → operation: CreateUser

⚠️ The XSD element names MUST follow this Request/Response naming convention
   for DefaultWsdl11Definition to auto-generate operations correctly.
```

### Complete Configuration:

```java
@Bean(name = "users")          // Bean name = WSDL filename → /ws/users.wsdl
public DefaultWsdl11Definition defaultWsdl11Definition(XsdSchema usersSchema) {

    DefaultWsdl11Definition wsdl = new DefaultWsdl11Definition();

    wsdl.setPortTypeName("UserPort");               // <portType name="UserPort">
    wsdl.setLocationUri("/ws");                     // <soap:address location="http://host/ws"/>
    wsdl.setTargetNamespace("http://example.com/user");
    wsdl.setSchema(usersSchema);

    // Optional: customize request/response suffix detection
    // wsdl.setRequestSuffix("Request");    // default
    // wsdl.setResponseSuffix("Response");  // default

    return wsdl;
}
```

### Verify Generated WSDL:

```bash
# Open in browser or curl
curl http://localhost:8080/ws/users.wsdl

# Or test with SoapUI:
# New Project → WSDL URL → http://localhost:8080/ws/users.wsdl → auto-generates test cases
```

> **Best Practice:** Use `DefaultWsdl11Definition` and let Spring WS generate the WSDL from XSD. Never write WSDL manually — it is error-prone and hard to maintain. The XSD is your only source of truth.

**Real World Analogy:** WSDL auto-generation is like **a restaurant menu being auto-printed from the kitchen's ingredient list**. You define what ingredients (XSD types) exist and what dishes are possible (Request/Response pairs). The menu (WSDL) is automatically created and handed to customers — you never type the menu manually.

---

## 21. Error Handling in SOAP?

**Error handling in SOAP** is done through **SOAP Faults** — a standardized XML error structure returned inside the SOAP Body when something goes wrong. Spring WS provides multiple mechanisms to map Java exceptions to SOAP Faults cleanly.

### Option 1 — `@SoapFault` Annotation on Custom Exception:

```java
// ==================== CUSTOM EXCEPTION WITH SOAP FAULT ====================
@SoapFault(faultCode = FaultCode.SERVER,
           faultStringOrReason = "User not found")
public class UserNotFoundException extends RuntimeException {

    public UserNotFoundException(String message) {
        super(message);
    }
}

// Usage in Endpoint — just throw the exception
@PayloadRoot(namespace = NAMESPACE_URI, localPart = "GetUserRequest")
@ResponsePayload
public GetUserResponse getUser(@RequestPayload GetUserRequest request) {
    User user = userService.findById(request.getUserId());

    if (user == null) {
        throw new UserNotFoundException("No user with ID: " + request.getUserId());
        // ✅ Spring WS auto-converts this to a proper SOAP Fault response
    }

    return buildResponse(user);
}
```

### Option 2 — Endpoint Interceptor for Global Fault Handling:

```java
// ==================== GLOBAL FAULT INTERCEPTOR ====================
@Component
public class SoapFaultInterceptor implements EndpointInterceptor {

    private static final Logger logger = LoggerFactory.getLogger(SoapFaultInterceptor.class);

    @Override
    public boolean handleFault(MessageContext messageContext, Object endpoint)
            throws Exception {

        // Log the fault details
        SaajSoapMessage response = (SaajSoapMessage) messageContext.getResponse();
        SOAPBody body = response.getSaajMessage().getSOAPBody();
        SOAPFault fault = body.getFault();

        if (fault != null) {
            logger.error("[SOAP FAULT] Code: {} | Message: {}",
                fault.getFaultCode(),
                fault.getFaultString());
        }

        return true;  // Allow fault response to be sent
    }

    // Other interface methods...
    @Override public boolean handleRequest(MessageContext ctx, Object ep) { return true; }
    @Override public boolean handleResponse(MessageContext ctx, Object ep) { return true; }
    @Override public void afterCompletion(MessageContext ctx, Object ep, Exception ex) {}
}
```

### Option 3 — Custom Exception Resolver:

```java
// ==================== CUSTOM SOAP FAULT RESOLVER ====================
@Component
public class CustomSoapFaultResolver
        extends AbstractEndpointExceptionResolver {

    @Override
    protected boolean resolveExceptionInternal(
            MessageContext messageContext,
            Object endpoint,
            Exception ex) {

        SaajSoapMessage response = (SaajSoapMessage) messageContext.getResponse();

        try {
            SOAPBody body = response.getSaajMessage().getSOAPBody();
            SOAPFault fault = body.addFault();

            if (ex instanceof UserNotFoundException) {
                fault.setFaultCode(new QName("CLIENT"));
                fault.setFaultString(ex.getMessage());
            } else if (ex instanceof ValidationException) {
                fault.setFaultCode(new QName("CLIENT"));
                fault.setFaultString("Invalid request: " + ex.getMessage());
            } else {
                fault.setFaultCode(new QName("SERVER"));
                fault.setFaultString("Internal server error");
            }

        } catch (SOAPException e) {
            // log
        }

        return true;  // Exception handled — send the fault
    }
}
```

**Real World Analogy:** SOAP error handling is like **a formal complaint process at a government office**. When something goes wrong, you don't get an angry shout — you receive an official complaint form (SOAP Fault) with a structured error code, reason, and details. The format is standardized so the recipient (client) knows exactly how to process the error programmatically.

---

## 22. What is SOAP Fault?

A **SOAP Fault** is the **standardized error response format in SOAP** — returned inside the `<Body>` element when a request cannot be processed successfully. It replaces the normal response payload with a structured error envelope that clients can parse programmatically.

| Element | Mandatory | Description |
|---|---|---|
| `<faultcode>` | ✅ Yes | Category of error — Client or Server |
| `<faultstring>` | ✅ Yes | Human-readable error description |
| `<faultactor>` | ❌ No | URI of the component that caused the fault |
| `<detail>` | ❌ No | Custom error detail — used for Body-related errors |

### Fault Codes:

| Code | Meaning | Cause |
|---|---|---|
| `soapenv:Client` | Client error | Bad request — wrong input, missing field, invalid data |
| `soapenv:Server` | Server error | Internal error — database down, NullPointerException |
| `soapenv:MustUnderstand` | Header not understood | Received `mustUnderstand="1"` header it can't process |
| `soapenv:VersionMismatch` | Wrong SOAP version | Client sent SOAP 1.2 to a SOAP 1.1 server |

### SOAP Fault XML:

```xml
<!-- SOAP Fault — returned in place of normal response -->
<soapenv:Envelope
    xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/">

    <soapenv:Body>
        <soapenv:Fault>

            <!-- ✅ Mandatory — fault category -->
            <faultcode>soapenv:Client</faultcode>

            <!-- ✅ Mandatory — human-readable message -->
            <faultstring>User not found with ID: 42</faultstring>

            <!-- ❌ Optional — who caused it (useful in intermediary chains) -->
            <faultactor>http://example.com/ws/UserService</faultactor>

            <!-- ❌ Optional — machine-readable detail (for Body-related faults) -->
            <detail>
                <errorCode>USER_NOT_FOUND</errorCode>
                <errorMessage>No user exists with the provided ID: 42</errorMessage>
                <timestamp>2024-01-15T10:30:00Z</timestamp>
            </detail>

        </soapenv:Fault>
    </soapenv:Body>

</soapenv:Envelope>
```

### Spring WS — Generate Fault Programmatically:

```java
// ==================== OPTION 1: @SoapFault on Exception class ====================
@SoapFault(
    faultCode = FaultCode.CLIENT,               // soapenv:Client
    faultStringOrReason = "Invalid user ID"
)
public class InvalidUserIdException extends RuntimeException {
    public InvalidUserIdException(String msg) { super(msg); }
}

// ==================== OPTION 2: SoapFaultMappingExceptionResolver in Config ====================
@Bean
public SoapFaultMappingExceptionResolver exceptionResolver() {
    SoapFaultMappingExceptionResolver resolver = new SoapFaultMappingExceptionResolver();

    SoapFaultDefinition defaultFault = new SoapFaultDefinition();
    defaultFault.setFaultCode(SoapFaultDefinition.SERVER);  // Default: SERVER fault
    resolver.setDefaultFault(defaultFault);

    // Map specific exceptions to specific fault types
    Properties mappings = new Properties();
    mappings.setProperty(UserNotFoundException.class.getName(),  "CLIENT");
    mappings.setProperty(ValidationException.class.getName(),    "CLIENT");
    mappings.setProperty(DatabaseException.class.getName(),      "SERVER");
    resolver.setExceptionMappings(mappings);

    resolver.setOrder(1);   // Highest priority
    return resolver;
}
```

### Fault Handling in Client:

```java
// ==================== HANDLING FAULT ON THE CLIENT SIDE ====================
try {
    GetUserResponse response = webServiceTemplate.marshalSendAndReceive(
        "http://localhost:8080/ws", request);

} catch (SoapFaultClientException ex) {
    // ✅ Spring WS wraps SOAP Faults in SoapFaultClientException
    String faultCode    = ex.getFaultCode().toString();
    String faultMessage = ex.getFaultStringOrReason();

    if ("Client".equals(faultCode)) {
        System.err.println("Bad request: " + faultMessage);
    } else {
        System.err.println("Server error: " + faultMessage);
    }
}
```

**Real World Analogy:** A SOAP Fault is like a **rejection letter from a government department**. Instead of ignoring your application, they send back a formal letter with a code (`faultcode` = type of rejection), reason (`faultstring` = why rejected), and details (`detail` = specific issues with your application). The format is standardized so every applicant (client) knows how to read and respond to the rejection.

---

*Happy Learning SOAP Web Services! 🌐*
