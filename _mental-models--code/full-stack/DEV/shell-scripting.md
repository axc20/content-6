# Shell Scripting

## API Example

```shell
BASE_URL="https://api.example.com"
EMAIL="your_email@example.com"
PASSWORD="your_password"
API_KEY=""
DATA_ENDPOINT="$BASE_URL/data"
STATUS_ENDPOINT="$BASE_URL/status"
CREATE_ENDPOINT="$BASE_URL/create"
DELETE_ENDPOINT="$BASE_URL/delete"

API_KEY=$(curl -s -X POST "$BASE_URL/auth" -d "email=$EMAIL&password=$PASSWORD" | jq -r '.api_key')

if [ -z "$API_KEY" ]; then
  echo "Failed to obtain API key"
  exit 1
else
  echo "API key obtained: $API_KEY"
fi

curl -X GET "$DATA_ENDPOINT" -H "Authorization: Bearer $API_KEY"
curl -X GET "$STATUS_ENDPOINT" -H "Authorization: Bearer $API_KEY"
curl -X POST "$CREATE_ENDPOINT" -H "Authorization: Bearer $API_KEY" -d '{"key": "value"}
curl -X DELETE "$DELETE_ENDPOINT/1" -H "Authorization: Bearer $API_KEY"

```
