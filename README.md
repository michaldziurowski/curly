# Curly - Curl Wrapper with Templates and Output Saving

Curly is a curl wrapper that adds templated variables and the ability to save response values for reuse in subsequent requests.

## Features

- 📝 **Template Variables**: Use `{{variable}}` syntax in URLs, headers, and request bodies
- 💾 **Save Response Values**: Extract and save values from responses using jq expressions in `.curly-state` file
- 🔄 **Variable Reuse**: Use saved values in subsequent requests through templates
- 🎨 **JSON Formatting**: Automatic JSON formatting with colored output
- **Simple Requirements**: Works with bash, curl, and jq

## Prerequisites

Curly requires the following tools to be installed:

- **bash** (version 4.0+)
- **curl** - for making HTTP requests
- **jq** - for parsing JSON responses

Check if you have them:

```bash
curl --version
jq --version
```

## Installation

1. Clone or download the curly script:

```bash
# Clone this repository
git clone https://github.com/michaldziurowski/curly.git

# Or download directly
wget https://raw.githubusercontent.com/michaldziurowski/curly/master/curly
chmod +x curly
```

2. Run curly from its directory:

```bash
cd curly
./curly --help
```

## Quick Start

### 1. Basic GET Request

```bash
./curly GET https://jsonplaceholder.typicode.com/users/1
```

### 2. Save Values from Response

```bash
# Get a user and save their ID and email
./curly GET https://jsonplaceholder.typicode.com/users/1 \
  -s 'user_id=.id' \
  -s 'user_email=.email'

# View saved variables
./curly --list
```

### 3. Use Saved Variables

```bash
# Use the saved user_id in the next request
./curly GET 'https://jsonplaceholder.typicode.com/users/{{user_id}}/posts'
```

## Usage Guide

### Command Syntax

```bash
./curly METHOD URL [OPTIONS]
```

#### HTTP Methods

- `GET`, `POST`, `PUT`, `DELETE`, `PATCH`, `HEAD`, `OPTIONS`

#### Options

| Option                  | Description              | Example                            |
| ----------------------- | ------------------------ | ---------------------------------- |
| `-d, --data <json>`     | Request body (JSON)      | `-d '{"name":"John"}'`             |
| `-H, --header <header>` | HTTP header (repeatable) | `-H 'Authorization: Bearer token'` |
| `-s, --save <var=jq>`   | Save response value      | `-s 'id=.data.id'`                 |
| `-o, --output <file>`   | Save response to file    | `-o response.json`                 |
| `-v, --verbose`         | Show debug information   | `-v`                               |
| `-q, --quiet`           | Suppress response output | `-q`                               |
| `--timeout <seconds>`   | Request timeout          | `--timeout 60`                     |
| `--no-format`           | Don't format JSON output | `--no-format`                      |

#### State Management

| Command               | Description                |
| --------------------- | -------------------------- |
| `--list`              | List all saved variables   |
| `--clear`             | Clear all saved variables  |
| `--set <var=value>`   | Manually set a variable    |
| `--unset <var>`       | Remove a specific variable |
| `--state-file <path>` | Use alternate state file   |

### Template Variables

Use `{{variable_name}}` syntax to reference saved values:

```bash
# In URLs
./curly GET 'https://api.example.com/users/{{user_id}}'

# In headers
./curly GET https://api.example.com/profile \
  -H 'Authorization: Bearer {{token}}'

# In request bodies
./curly POST https://api.example.com/posts \
  -d '{"author_id": {{user_id}}, "title": "Hello"}'
```

### Saving Values with JQ Expressions

Curly uses [jq](https://stedolan.github.io/jq/) expressions to extract values from JSON responses:

```bash
# Save simple values
-s 'variable=.field'

# Save nested values
-s 'token=.auth.access_token'

# Save array elements
-s 'first_user=.users[0].id'

# Save with filters
-s 'user_count=.users | length'

# Save with conditions
-s 'active_user=.users[] | select(.status=="active") | .id'
```

### State File

Variables are stored in `./.curly-state` in the current directory:

```bash
# Default location
./.curly-state

# Custom location
./curly GET https://api.example.com --state-file ~/my-api-state

# View state file directly
cat .curly-state
```

Format:

```
token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
user_id=42
api_base=https://api.example.com
last_http_code=200
```

## Real-World Examples

### Authentication Flow

```bash
# 1. Login and save token
./curly POST https://api.example.com/auth/login \
  -d '{"username":"admin","password":"secret"}' \
  -s 'token=.access_token' \
  -s 'refresh=.refresh_token' \
  -s 'expires_in=.expires_in'

# 2. Use token in subsequent requests
./curly GET https://api.example.com/user/profile \
  -H 'Authorization: Bearer {{token}}'

# 3. Update profile
./curly PATCH https://api.example.com/user/profile \
  -H 'Authorization: Bearer {{token}}' \
  -d '{"email":"newemail@example.com"}'
```

### CRUD Operations

```bash
# Create a resource
./curly POST https://api.example.com/projects \
  -H 'Authorization: Bearer {{token}}' \
  -d '{"name":"New Project","description":"Description"}' \
  -s 'project_id=.id'

# Read the resource
./curly GET 'https://api.example.com/projects/{{project_id}}' \
  -H 'Authorization: Bearer {{token}}'

# Update the resource
./curly PUT 'https://api.example.com/projects/{{project_id}}' \
  -H 'Authorization: Bearer {{token}}' \
  -d '{"name":"Updated Project"}'

# Delete the resource
./curly DELETE 'https://api.example.com/projects/{{project_id}}' \
  -H 'Authorization: Bearer {{token}}'
```

### Pagination

```bash
# First page
./curly GET 'https://api.example.com/items?page=1&limit=10' \
  -H 'Authorization: Bearer {{token}}' \
  -s 'next_page=.pagination.next' \
  -s 'total_pages=.pagination.total'

# Next page
./curly GET 'https://api.example.com/items?page={{next_page}}&limit=10' \
  -H 'Authorization: Bearer {{token}}' \
  -s 'next_page=.pagination.next'
```

### Working with Multiple Environments

```bash
# Development environment
./curly POST https://dev.api.example.com/login \
  --state-file .curly-state-dev \
  -d '{"username":"dev_user","password":"dev_pass"}' \
  -s 'token=.token'

# Production environment
./curly POST https://api.example.com/login \
  --state-file .curly-state-prod \
  -d '{"username":"prod_user","password":"prod_pass"}' \
  -s 'token=.token'
```

### Complex Workflow Example

```bash
#!/bin/bash
# Script: setup-project.sh

# Clear any previous state
./curly --clear

# 1. Authenticate
echo "Authenticating..."
./curly POST https://api.example.com/auth/login \
  -d '{"username":"admin","password":"secret"}' \
  -s 'token=.token' \
  -s 'user_id=.user.id' \
  -q

# 2. Create project
echo "Creating project..."
./curly POST https://api.example.com/projects \
  -H 'Authorization: Bearer {{token}}' \
  -d '{"name":"My Project","owner_id":{{user_id}}}' \
  -s 'project_id=.id' \
  -q

# 3. Add team members
echo "Adding team members..."
./curly POST 'https://api.example.com/projects/{{project_id}}/members' \
  -H 'Authorization: Bearer {{token}}' \
  -d '{"email":"alice@example.com","role":"developer"}' \
  -q

./curly POST 'https://api.example.com/projects/{{project_id}}/members' \
  -H 'Authorization: Bearer {{token}}' \
  -d '{"email":"bob@example.com","role":"designer"}' \
  -q

# 4. Create initial tasks
echo "Creating tasks..."
./curly POST 'https://api.example.com/projects/{{project_id}}/tasks' \
  -H 'Authorization: Bearer {{token}}' \
  -d '{"title":"Setup repository","assignee":"alice@example.com"}' \
  -s 'task1_id=.id' \
  -q

./curly POST 'https://api.example.com/projects/{{project_id}}/tasks' \
  -H 'Authorization: Bearer {{token}}' \
  -d '{"title":"Create mockups","assignee":"bob@example.com"}' \
  -s 'task2_id=.id' \
  -q

echo "Project setup complete!"
./curly --list
```

## Advanced Usage

### Debugging Requests

Use verbose mode to see detailed information:

```bash
./curly GET https://api.example.com/users -v
```

This shows:

- Template replacements
- Full curl command
- HTTP status codes
- Variable saves

### Handling Errors

Error messages:

```bash
# Missing variable
./curly GET 'https://api.example.com/users/{{unknown_var}}'
# Error: Variable 'unknown_var' not found. Use --list to see available variables.

# Invalid JSON
./curly POST https://api.example.com/users -d 'invalid-json'
# Warning: HTTP 400 response
# Error response: {"error":"Invalid JSON"}

# Failed extraction
./curly GET https://api.example.com/users -s 'id=.nonexistent'
# Warning: Expression '.nonexistent' returned null or empty value
```

### Using with Other Tools

Pipe curly output to other commands:

```bash
# Pretty print with less
./curly GET https://api.example.com/data | less

# Extract specific fields with jq
./curly GET https://api.example.com/users --no-format | jq '.users[].email'

# Save formatted output
./curly GET https://api.example.com/data | jq '.' > formatted.json

# Count results
./curly GET https://api.example.com/items --quiet | jq '.items | length'
```

### Scripting with Curly

Example bash script using curly:

```bash
#!/bin/bash

# Load environment-specific config
if [[ "$ENV" == "production" ]]; then
    API_BASE="https://api.example.com"
    STATE_FILE=".curly-state-prod"
else
    API_BASE="https://dev.api.example.com"
    STATE_FILE=".curly-state-dev"
fi

# Set base URL
./curly --set "api_base=$API_BASE" --state-file "$STATE_FILE"

# Function to make authenticated requests
api_request() {
    local method="$1"
    local endpoint="$2"
    shift 2

    ./curly "$method" "{{api_base}}$endpoint" \
        -H 'Authorization: Bearer {{token}}' \
        --state-file "$STATE_FILE" \
        "$@"
}

# Login
./curly POST "{{api_base}}/auth/login" \
    --state-file "$STATE_FILE" \
    -d '{"username":"user","password":"pass"}' \
    -s 'token=.token'

# Now use the helper function
api_request GET /users
api_request POST /projects -d '{"name":"New Project"}'
```

## Tips and Best Practices

1. **Organize State Files**: Use different state files for different environments or projects

   ```bash
   ./curly GET api.com --state-file .curly-dev
   ./curly GET api.com --state-file .curly-prod
   ```

2. **Clear State Regularly**: Clear variables when starting new workflows

   ```bash
   ./curly --clear
   ```

3. **Use Verbose Mode for Debugging**: Add `-v` when troubleshooting

   ```bash
   ./curly GET api.com -v
   ```

4. **Save Complete Responses**: Use `-o` to save full responses for reference

   ```bash
   ./curly GET api.com -o response.json
   ```

5. **Chain Commands**: Use `&&` to chain dependent requests

   ```bash
   ./curly POST api.com/login -s 'token=.token' && \
   ./curly GET api.com/profile -H 'Auth: {{token}}'
   ```

6. **Validate Responses**: Check HTTP status codes

   ```bash
   ./curly GET api.com && echo "Success!" || echo "Failed!"
   ```

7. **Use Shell Scripts**: Wrap complex workflows in scripts for reusability

## Troubleshooting

### Common Issues

**Problem**: "Missing required dependencies: curl jq"

- **Solution**: Install the missing tools using your package manager

**Problem**: "Variable 'xxx' not found"

- **Solution**: Check available variables with `./curly --list`
- Ensure the variable was saved in a previous request

**Problem**: "Failed to extract value with expression"

- **Solution**: Test your jq expression separately:
  ```bash
  ./curly GET api.com -o test.json
  jq '.your.expression' test.json
  ```

**Problem**: "HTTP 401 Unauthorized"

- **Solution**: Check if your token is expired
- Verify the Authorization header format

**Problem**: State not persisting between calls

- **Solution**: Ensure you're running curly from the same directory
- Check if `.curly-state` file exists and is readable

### Debug Mode

Run with `-v` for detailed output:

```bash
./curly GET https://api.example.com -v
```

This shows:

- Variable loading
- Template replacements
- Full curl command
- Response processing
- Variable saves

## Security Considerations

1. **State File Security**: The `.curly-state` file may contain sensitive data like tokens

   ```bash
   # Add to .gitignore
   echo ".curly-state*" >> .gitignore

   # Set restrictive permissions
   chmod 600 .curly-state
   ```

2. **Token Management**: Consider token expiration

   ```bash
   # Save token expiry time
   ./curly POST api.com/login \
     -s 'token=.token' \
     -s 'token_expires=.expires_at'
   ```

3. **Secure Transmission**: Always use HTTPS for sensitive data

4. **Environment Variables**: For sensitive data, consider using environment variables
   ```bash
   ./curly POST api.com/login \
     -d "{\"password\":\"$API_PASSWORD\"}"
   ```
