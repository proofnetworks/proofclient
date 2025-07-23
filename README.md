# ProofNetwork Client Library

A comprehensive TypeScript client library for integrating with ProofNetwork smart contracts and content API.

## Features

- **Type-safe contract calls** with automatic validation
- **Wallet integration** with Phantom and other Solana wallets
- **Authentication management** with session tokens
- **Rate limiting** and request queuing
- **Circuit breaker** pattern for resilience
- **Content API** with caching and ETags
- **Retry logic** with exponential backoff
- **Response parsing** with schema validation

## Installation



### Browser (Local)

```html
<script src="path/to/proofnetwork-client.min.js"></script>
```

## Quick Start

### TypeScript / ES Modules

```typescript
import { ProofNetworkClient } from '@proofnetwork/client';

// Initialize the client
const client = new ProofNetworkClient({
  apiUrl: 'https://proofnetwork.com',
  timeout: 30000
});

// Initialize and authenticate
await client.initialize();
await client.authenticate();

// Call a contract function
const result = await client.quickCall(
  'your-contract-id',
  'your-function-name',
  { param1: 'value1', param2: 42 }
);

console.log(result);
```

### Browser

```html
<script src="https://unpkg.com/@proofnetwork/client/dist/proofnetwork-client.min.js"></script>
<script>
  // The library is available as ProofNetworkClient global
  const client = new ProofNetworkClient.ProofNetworkClient({
    apiUrl: 'https://proofnetwork.com',
    timeout: 30000
  });

  // Initialize and use
  client.initialize().then(async () => {
    await client.authenticate();
    
    const result = await client.quickCall(
      'your-contract-id',
      'your-function-name',
      { param1: 'value1', param2: 42 }
    );
    
    console.log(result);
  });
</script>
```

## Authentication

The client uses wallet-based authentication with challenge-response flow:

```typescript
// Listen for authentication state changes
client.onAuthenticationStateChange((state) => {
  console.log('Authenticated:', state.isAuthenticated);
  console.log('Session expires:', new Date(state.expiresAt));
});

// Authenticate (will prompt wallet connection)
try {
  const authState = await client.authenticate();
  console.log('Successfully authenticated:', authState);
} catch (error) {
  console.error('Authentication failed:', error.message);
}

// Logout
await client.logout();
```

## Contract Calls

### Basic Contract Calls

```typescript
// Simple contract call
const response = await client.callContract({
  contractId: 'arena-combat-v1',
  functionName: 'joinArena',
  parameters: { playerClass: 'archer' },
  gasLimit: 100000
});

if (response.success) {
  console.log('Joined arena:', response.data);
} else {
  console.error('Failed to join:', response.error);
}
```

### Batch Contract Calls

```typescript
const requests = [
  {
    contractId: 'arena-combat-v1',
    functionName: 'getPlayerStats',
    parameters: { playerId: 'player1' }
  },
  {
    contractId: 'arena-combat-v1',
    functionName: 'getPlayerStats',
    parameters: { playerId: 'player2' }
  }
];

const responses = await client.batchCallContracts(requests);
responses.forEach((response, index) => {
  console.log(`Player ${index + 1}:`, response.data);
});
```

### Safe Contract Calls

```typescript
// Safe call that won't throw errors
const result = await client.safeCall('contract-id', 'function-name', {
  param: 'value'
});

if (result.success) {
  console.log('Data:', result.data);
} else {
  console.error('Error:', result.error);
}
```

## Content API

```typescript
// Get content with caching
const content = await client.getContent('/game-data/levels.json');
console.log('Level data:', content.data);

// Update content (requires authentication)
await client.updateContent('/game-data/scores.json', {
  topScores: [1000, 900, 800]
});

// List content in a directory
const listing = await client.getContent('/assets/', {
  recursive: true,
  includeMetadata: true
});
```

## Error Handling

```typescript
import { 
  AuthenticationError, 
  RateLimitError, 
  ContractCallError 
} from '@proofnetwork/client';

try {
  await client.callContract({ /* ... */ });
} catch (error) {
  if (error instanceof AuthenticationError) {
    console.log('Need to authenticate first');
    await client.authenticate();
  } else if (error instanceof RateLimitError) {
    console.log('Rate limited, retrying in:', error.details?.retryAfter);
  } else if (error instanceof ContractCallError) {
    console.log('Contract call failed:', error.message);
  }
}
```

## Configuration

```typescript
const client = new ProofNetworkClient({
  apiUrl: 'https://proofnetwork.com',
  timeout: 30000,
  sessionToken: 'existing-token', // Optional: use existing session
  
  // Retry configuration
  retryConfig: {
    strategy: 'exponential',
    maxRetries: 3,
    baseDelay: 1000,
    maxDelay: 10000,
    jitter: true
  },
  
  // Circuit breaker configuration
  circuitBreakerConfig: {
    failureThreshold: 5,
    recoveryTimeout: 30000,
    monitoringPeriod: 60000
  },
  
  // Rate limiting configuration
  rateLimitConfig: {
    queueProcessingInterval: 1000,
    maxQueueSize: 1000,
    defaultBackoffMs: 1000
  }
});
```

## Schema Validation

Register schemas for automatic response validation:

```typescript
// Register a schema
client.registerResponseSchema('PlayerStats', {
  type: 'object',
  properties: {
    health: { type: 'number', minimum: 0, maximum: 100 },
    level: { type: 'number', minimum: 1 },
    class: { type: 'string', pattern: '^(archer|rogue|gladiator|mage)$' }
  },
  required: ['health', 'level', 'class']
});

// Use schema validation
const response = await client.callContract({
  contractId: 'game-v1',
  functionName: 'getPlayerStats',
  parameters: { playerId: '123' }
}, 'PlayerStats'); // Schema name
```

## Client Status

```typescript
const status = client.getClientStatus();
console.log('Status:', {
  initialized: status.isInitialized,
  authenticated: status.isAuthenticated,
  queueLength: status.rateLimit.queueLength,
  circuitBreaker: status.circuitBreaker
});
```

## Cleanup

```typescript
// Clean up resources when done
client.destroy();
```

## TypeScript Support

The library is fully typed with comprehensive TypeScript definitions:

```typescript
import { 
  ProofNetworkClient,
  ContractCallRequest,
  ContractCallResponse,
  AuthenticationState,
  ProofNetworkConfig
} from '@proofnetwork/client';

// All interfaces and types are exported
```

## License

MIT License - see LICENSE file for details.