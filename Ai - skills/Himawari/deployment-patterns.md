# Deployment Patterns: Himawari

## GRAPH METADATA
- cluster: himawari
- node_type: skill
- importance_level: medium
- hub_node: false
- tags: [deployment, build, coding-patterns, skills]

## NODE IMPORTANCE SCORING
- importance_score: 0.8
- hub_score: 0.2
- reuse_score: 0.7
- stability_score: 0.95
- dependency_weight: 0.4

## GRAPH VISUALIZATION
```mermaid
graph TD
    SDeploy[Ai - skills/Himawari/deployment-patterns.md] --> Deploy[[[12-deployment.md]]]
```

This document outlines deployment automation configurations, discloud templates, and mod file deployment patterns.

## Discloud Deployment Configuration

Create a `discloud.config` in the root of the Discord Bot folder to configure the Discloud runtime environment:

```ini
NAME=Himawari
TYPE=bot
MAIN=index.js
RAM=100
VERSION=latest
AUTORESTART=true
```

## Fabric Mod Deployment Automation

To streamline manual testing of Fabric mod files, configure a gradle task in `build.gradle` to copy compiled mod JARs directly into a local Minecraft server's mods directory:

```groovy
def modsDir = file(project.findProperty('modsDir') ?: layout.buildDirectory.dir('mods-deploy').get())

tasks.register('deployToMods', Copy) {
    dependsOn tasks.named('jar')
    from tasks.named('jar')
    into modsDir
    onlyIf { modsDir.exists() }
    
    // Prune previous mod builds
    doFirst {
        delete fileTree(modsDir) {
            include "${project.archives_base_name}-*.jar"
        }
    }
}

tasks.named('build') {
    finalizedBy 'deployToMods'
}
```

Add your target destination in `gradle.properties`:
```properties
modsDir=C:/path/to/server/mods
```

## FUTURE ARCHITECTURE PREDICTION

### 1. NEXT STATE ARCHITECTURE
* Transition to deploying Node bot dependencies using lightweight Docker configurations on serverless orchestrators like Fly.io or Railway.
* Automate jar delivery to production servers using secure SFTP Gradle plugins.
