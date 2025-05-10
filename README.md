# AWS Cognito Lambda Triggers

This project demonstrates the implementation of AWS Cognito with Lambda triggers for user authentication workflows. It uses AWS SAM (Serverless Application Model) to deploy a complete authentication system with customizable pre-signup and post-confirmation Lambda functions.

<!-- ## Architecture

![Architecture Diagram](https://d1.awsstatic.com/product-marketing/Lambda/Diagrams/product-page-diagram_Lambda-RealTimeFileProcessing.a59577de4b6471674a540b878b0b684e0249a18c.png) -->

The project consists of:

1. **AWS Cognito User Pool** - Manages user authentication and authorization
2. **Lambda Triggers**:
   - Pre-Signup Lambda - Executes before a user signs up
   - Post-Confirmation Lambda - Executes after a user confirms their account
3. **API Gateway HTTP API** - Protected by JWT authentication from Cognito
4. **Application Lambda** - Sample application logic

## Prerequisites

- [AWS CLI](https://aws.amazon.com/cli/) installed and configured
- [AWS SAM CLI](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-install.html) installed
- Node.js 22.x or later

## Project Structure

```
aws-cognito-lambda-triggers/
├── app/                      # Application Lambda function
│   └── app.js                # Simple handler that returns the event
├── postconfirmation/         # Post-confirmation Lambda trigger
│   └── postConfirmation.mjs  # Handler for post-confirmation events
├── presignup/                # Pre-signup Lambda trigger
│   └── preSignUp.mjs         # Handler for pre-signup events
├── .gitignore                # Git ignore file
├── cognito.yaml              # Cognito resources CloudFormation template
├── README.md                 # Project documentation
├── samconfig.toml            # SAM CLI configuration
├── sample.samconfig.toml     # Sample SAM configuration
└── template.yaml             # Main CloudFormation template
```

## Setup and Deployment

### Configuration

1. Copy the sample configuration file:
   ```bash
   cp sample.samconfig.toml samconfig.toml
   ```

2. Edit `samconfig.toml` to set your specific parameters:
   - `stack_name`: Name of your CloudFormation stack
   - `region`: AWS region to deploy to
   - `profile`: AWS CLI profile to use
   - `parameter_overrides`: Configure the following parameters:
     - `Client`: Your application domain (e.g., "https://myapp.com")
     - `TestWithPostman`: Set to "true" if testing with Postman
     - `CognitoARN`: ARN of your existing Cognito User Pool (if applicable)

### Deployment

Deploy the application using SAM CLI:

```bash
sam deploy
```

After deployment, SAM will output:
- API Gateway endpoint URL
- Cognito authentication URL
- User Pool Client ID

## Lambda Triggers

### Pre-Signup Trigger

Located in `presignup/preSignUp.mjs`, this Lambda function executes before a user signs up. You can customize this function to:

- Automatically confirm users from specific domains
- Implement custom validation logic
- Block signups based on custom criteria

Example implementation for auto-confirming users:

```javascript
export const handler = async (event) => {
  // Auto confirm users
  event.response.autoConfirmUser = true;
  
  // Auto verify email
  if (event.request.userAttributes.hasOwnProperty("email")) {
    event.response.autoVerifyEmail = true;
  }
  
  return event;
};
```

### Post-Confirmation Trigger

Located in `postconfirmation/postConfirmation.mjs`, this Lambda function executes after a user confirms their account. You can customize this function to:

- Create user records in a database
- Send welcome emails
- Assign default roles or permissions

Example implementation for creating a user record:

```javascript
import { DynamoDBClient } from "@aws-sdk/client-dynamodb";
import { DynamoDBDocumentClient, PutCommand } from "@aws-sdk/lib-dynamodb";

const client = new DynamoDBClient({});
const docClient = DynamoDBDocumentClient.from(client);

export const handler = async (event) => {
  if (event.triggerSource === "PostConfirmation_ConfirmSignUp") {
    const { email, sub } = event.request.userAttributes;
    
    try {
      await docClient.send(
        new PutCommand({
          TableName: "Users",
          Item: {
            userId: sub,
            email: email,
            createdAt: new Date().toISOString()
          }
        })
      );
      console.log(`User ${email} added to database`);
    } catch (error) {
      console.error("Error adding user to database:", error);
    }
  }
  
  return event;
};
```

## Testing

### Testing with Postman

If you've set `TestWithPostman` to "true" in your deployment:

1. Use the Cognito authentication URL from the deployment outputs
2. Configure Postman OAuth 2.0 authorization:
   - Auth URL: `{AuthUrl}/oauth2/authorize`
   - Access Token URL: `{AuthUrl}/oauth2/token`
   - Client ID: Use the Client ID from deployment outputs
   - Scope: `email openid profile`
   - Grant Type: Authorization Code or Implicit (for testing)

### Testing with a Web Application

1. Configure your web application to use the Cognito User Pool
2. Use the Client ID from deployment outputs
3. Set the callback URL to match what you configured in the deployment

## Customization

### Modifying Lambda Triggers

Edit the Lambda trigger functions in their respective directories:
- `presignup/preSignUp.mjs`
- `postconfirmation/postConfirmation.mjs`

After making changes, redeploy using:

```bash
sam deploy
```

### Adding Additional Triggers

To add more Cognito Lambda triggers:

1. Create a new directory for your trigger
2. Add the Lambda function code
3. Update `template.yaml` to include the new function and permissions
4. Uncomment and update the `LambdaConfig` section in `cognito.yaml`

## Security Considerations

- Use environment variables for sensitive information
- Follow the principle of least privilege for IAM roles
- Enable MFA for Cognito users in production
- Use HTTPS for all endpoints
- Implement proper error handling in Lambda functions

## Resources

- [AWS Cognito Documentation](https://docs.aws.amazon.com/cognito/)
- [Lambda Triggers for Cognito](https://docs.aws.amazon.com/cognito/latest/developerguide/cognito-user-identity-pools-working-with-aws-lambda-triggers.html)
- [AWS SAM Documentation](https://docs.aws.amazon.com/serverless-application-model/)

## License

This project is licensed under the MIT-0 License.