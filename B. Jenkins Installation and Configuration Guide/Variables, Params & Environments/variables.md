## In Jenkins declarative pipelines, the correct keywords inside parameters {} are:

string(...)

choice(...)

booleanParam(...) (not just boolean)

### 🔹 Correct Examples
```sh
String
parameters {
  string(name: 'DEPLOY_ENV', defaultValue: 'dev', description: 'Environment to deploy to')
}
```

Choice
```sh
parameters {
  choice(name: 'BUILD_TYPE', choices: ['debug', 'release'], description: 'Build profile')
}
```

Boolean
```sh
parameters {
  booleanParam(name: 'RUN_TESTS', defaultValue: true, description: 'Run tests?')
}
```

### 🔍 Why this matters
- ✅ booleanParam is the official DSL keyword.
- ❌ Writing boolean will cause a syntax error in Jenkinsfile.