---
name: cli-architecture-pattern
description: Structured CLI architecture using 2-tier or 3-tier patterns with defined entrypoints, manager classes, and reusable commands. Apply when creating CLI tools, scripts, or any command-line interface instead of ad-hoc scripts.
type: pattern
---

# CLI Architecture Pattern

**This is a reference pattern.** Learn from the approach, adapt to your context — don't copy verbatim.

**Problem**: Infrastructure operations scattered across shell scripts, manual commands, and deployment pipelines become hard to maintain and inconsistent.

**Solution**: Structured CLI tools with uniform entry points.

---

## Why This Pattern?

**Benefits**:
- **Consistency**: Same commands work locally, in CI/CD, and deployment scripts
- **Readability**: `pnpm deploy:dev` is clearer than raw AWS CLI commands
- **Testability**: CLI commands can be tested independently
- **Reusability**: Share logic across different entry points
- **Documentation**: Commands are self-documenting with `--help`

**Integration Points**:
- Root `package.json` scripts (JS/TS projects)
- `pyproject.toml` scripts (Python projects)
- GitHub Actions workflows
- Deployment scripts (bash, etc.)
- Local development

**Example Integration**:
```json
// package.json
{
  "scripts": {
    "deploy:dev": "ts-node cli/bin/deploy.ts --env=dev",
    "export-params:dev": "ts-node cli/bin/ssm-params.ts export dev frontend"
  }
}
```

```yaml
# GitHub Actions
- name: Deploy infrastructure
  run: pnpm deploy:dev

- name: Export parameters
  run: pnpm export-params:dev
```

---

## 1-Tier Architecture (For Simple Projects)

**Structure**:
```
lib/
  infra.ts               # Single entry point: arg parsing + business logic
  core/
    environment-manager.ts  # Bootstrap config loader (shared utility)
```

**Single File** - All commands in one switch:
```typescript
// lib/infra.ts
import { EnvironmentManager } from './core/environment-manager';
import { S3Client, PutObjectCommand } from '@aws-sdk/client-s3';
import { CloudFormationClient, DescribeStacksCommand } from '@aws-sdk/client-cloudformation';
import { CloudFrontClient, CreateInvalidationCommand } from '@aws-sdk/client-cloudfront';
import { spawn } from 'child_process';

const [command] = process.argv.slice(2);

async function main() {
  switch (command) {
    case 'deploy':
      await deploy();
      break;
    case 'publish':
      await publish();
      break;
    case 'synth':
      await run('npx', ['cdk', 'synth']);
      break;
    default:
      console.log('Commands: deploy | publish | synth');
      process.exit(1);
  }
}

async function deploy() {
  console.log('📦 Provisioning infrastructure...');
  await run('npx', ['cdk', 'deploy', '--all', '--require-approval', 'never']);
  console.log('🏗️ Publishing frontend...');
  await publish();
}

async function publish() {
  const stage = getStage();
  const stackName = `myproject-web-${stage}`;
  const region = new EnvironmentManager(stage).load().cdkDefaultRegion;

  await run('pnpm', ['--filter', 'frontend', 'run', 'build']);

  const bucketName = await getStackOutput(stackName, 'WebBucketName', region);
  await syncToS3('frontend/out', bucketName, region);

  const distId = await getStackOutput(stackName, 'CloudfrontDistributionId', region);
  await invalidate(distId, region);

  console.log('✅ Published');
}

// ... helper functions (run, getStackOutput, syncToS3, invalidate)

main().catch((err) => {
  console.error(err.message);
  process.exit(1);
});
```

**Integration**:
```json
// package.json
{
  "scripts": {
    "deploy:dev": "ENVIRONMENT=dev ts-node infrastructure/lib/infra.ts deploy",
    "deploy:prod": "ENVIRONMENT=prod ts-node infrastructure/lib/infra.ts deploy",
    "publish:dev": "ENVIRONMENT=dev ts-node infrastructure/lib/infra.ts publish",
    "synth": "ENVIRONMENT=dev ts-node infrastructure/lib/infra.ts synth"
  }
}
```

**When to Use**:
- Simple projects (static frontends, single-stack)
- ≤5 commands
- No shared logic between commands worth abstracting
- Fast implementation preferred

**When to Migrate to 2-Tier**:
- Commands grow beyond ~200 lines total
- You need `--verbose`, `--dry-run`, or other shared flags
- Multiple commands share significant logic

---

## 2-Tier Architecture (Recommended for Most Projects)

**Structure**:
```
cli/
  bin/           # Tier 1: Argument parsing
  commands/      # Tier 2: Business logic + SDK calls
shared/
  env-loader.ts  # Simple utilities
  configs.ts
```

**Tier 1: CLI Binaries** - Entry points only
```typescript
// bin/ssm-params.ts
import { Command } from 'commander';
import { getParameter } from '../commands/ssm-params';

const program = new Command();

program
  .command('get <name>')
  .action(async (name: string) => {
    const value = await getParameter(name);
    console.log(value);
  });

program.parse();
```

**Tier 2: Commands** - Direct SDK usage
```typescript
// commands/ssm-params.ts
import { SSMClient, GetParameterCommand } from '@aws-sdk/client-ssm';
import { loadEnv } from '../shared/env-loader';

loadEnv();

const ssmClient = new SSMClient({ 
  region: process.env.CDK_DEFAULT_REGION 
});

export async function getParameter(name: string): Promise<string> {
  const command = new GetParameterCommand({ Name: name });
  const response = await ssmClient.send(command);
  return response.Parameter?.Value ?? '';
}
```

**When to Use**:
- Medium-sized projects
- 2-5 CLI tools
- Straightforward operations
- Faster development preferred

---

## 3-Tier Architecture (For Complex Projects)

**Structure**:
```
cli/
  bin/           # Tier 1: Argument parsing + error handling ONLY
  commands/      # Tier 2: Business logic + manager instantiation
core/
  base-manager.ts    # Tier 3: Shared abstractions
  aws-manager.ts
  env-manager.ts
```

**Tier 1: CLI Binaries** - Entry points only (NO business logic)
```typescript
// bin/ssm-params.ts
import { Command } from 'commander';
import { SSMParamsCommand } from '../commands/ssm-params';

const program = new Command();

program
  .command('get <name>')
  .action(async (name: string) => {
    // ONLY: Parse arguments, instantiate command, call method
    const cmd = new SSMParamsCommand();
    const value = await cmd.getParameter(name);
    console.log(value);
  });

program.parse();
```

**Tier 2: Commands** - ALL business logic (orchestration, config, managers)
```typescript
// commands/ssm-params.ts
import { AWSManager } from '../core/aws-manager';
import { EnvironmentManager } from '../core/env-manager';

export class SSMParamsCommand {
  private awsManager: AWSManager;
  private envManager: EnvironmentManager;

  constructor() {
    // Command instantiates and configures managers
    this.envManager = new EnvironmentManager();
    this.awsManager = new AWSManager(this.envManager);
  }

  async getParameter(name: string): Promise<string> {
    // Business logic: retrieve config, call managers
    const region = this.envManager.getRegion();
    return await this.awsManager.getSSMParameter(name, region);
  }
}
```

**Tier 3: Core Managers** - Reusable abstractions

**BaseManager** - Common functionality:
```typescript
// core/base-manager.ts
export class BaseManager {
  protected verbose: boolean;

  constructor(config: { verbose?: boolean } = {}) {
    this.verbose = config.verbose ?? false;
  }

  protected log(message: string): void {
    if (this.verbose) console.log(message);
  }

  protected validateRequired(value: any, name: string): void {
    if (!value) {
      throw new Error(`${name} is required`);
    }
  }
}
```

**Specific Managers** - Inherit from BaseManager:
```typescript
// core/env-manager.ts
import { BaseManager } from './base-manager';

export class EnvironmentManager extends BaseManager {
  loadEnv(stage: string): Record<string, string> {
    this.log(`Loading environment for ${stage}`);
    return process.env;
  }

  getRegion(): string {
    this.validateRequired(process.env.AWS_REGION, 'AWS_REGION');
    return process.env.AWS_REGION!;
  }
}
```

```typescript
// core/aws-manager.ts
import { SSMClient, GetParameterCommand } from '@aws-sdk/client-ssm';
import { BaseManager } from './base-manager';
import { EnvironmentManager } from './env-manager';

export class AWSManager extends BaseManager {
  private envManager: EnvironmentManager;
  private ssmClient: SSMClient;

  constructor(envManager: EnvironmentManager) {
    super();
    this.envManager = envManager;
    this.ssmClient = new SSMClient({ 
      region: envManager.getRegion() 
    });
  }

  async getSSMParameter(name: string, region: string): Promise<string> {
    this.log(`Fetching parameter: ${name}`);
    const command = new GetParameterCommand({ Name: name });
    const response = await this.ssmClient.send(command);
    return response.Parameter?.Value ?? '';
  }
}
```

**When to Use**:
- Large projects with 5+ CLI tools
- Complex operations across multiple services
- Team environments needing consistency
- Reusable abstractions provide clear value

---

## Decision Guide

| Factor | 1-Tier | 2-Tier | 3-Tier |
|--------|--------|--------|--------|
| Project Size | Simple | Medium | Large |
| CLI Commands | ≤5 | 2-5 | 5+ |
| Team Size | 1 | 1-3 | 3+ |
| Complexity | Minimal | Moderate | Complex |
| Development Speed | Fastest | Fast | Slower initially |
| Reusability | None | Lower | Higher |
| Shared Logic | Minimal | Some | Significant |

**Start with 1-Tier** for static frontends and single-stack projects.
**Migrate to 2-Tier** when commands exceed ~200 lines or share significant logic.
**Migrate to 3-Tier** when:
- You have 5+ CLI tools
- Duplicating logic across commands
- Team needs consistent patterns
- Abstractions provide clear value

---

## Language-Specific Integration

**JavaScript/TypeScript** - `package.json`:
```json
{
  "scripts": {
    "deploy:dev": "ts-node cli/bin/deploy.ts --env=dev",
    "params:get": "ts-node cli/bin/ssm-params.ts get"
  }
}
```

**Python** - `pyproject.toml`:
```toml
[tool.poetry.scripts]
deploy-dev = "cli.bin.deploy:main"
params-get = "cli.bin.ssm_params:get_parameter"
```

**Makefile** - Universal:
```makefile
deploy-dev:
	python cli/bin/deploy.py --env=dev

params-get:
	python cli/bin/ssm_params.py get $(NAME)
```

---

## Related Patterns

- [Pure Functions](../../CORE_PRINCIPLES.md#1-prefer-pure-functions) - Apply to helper functions
- [Error Handling](../../CORE_PRINCIPLES.md#7-error-handling) - Fail fast on configuration errors
