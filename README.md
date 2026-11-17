# jinahyaee-bom

Bills of materials of the `jinahya-*` artifacts, one per Jakarta EE generation, as
`jakarta.platform:jakarta.jakartaee-bom` is for the `jakarta.*` APIs.

| Jakarta EE | Bill of materials | Manages |
| --- | --- | --- |
| 10 | `io.github.jinahya:jinahyaee-bom-10` | `jinahya-validation-*-10` |
| 11 | `io.github.jinahya:jinahyaee-bom-11` | `jinahya-persistence-*-11`, `jinahya-validation-*-11` |

Import the one of your generation beside the Jakarta EE BOM, and declare the artifacts without versions:

```xml
<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>jakarta.platform</groupId>
      <artifactId>jakarta.jakartaee-bom</artifactId>
      <version>11.0.0</version>
      <type>pom</type>
      <scope>import</scope>
    </dependency>
    <dependency>
      <groupId>io.github.jinahya</groupId>
      <artifactId>jinahyaee-bom-11</artifactId>
      <version>${version.io.github.jinahya.jinahyaee-bom}</version>
      <type>pom</type>
      <scope>import</scope>
    </dependency>
  </dependencies>
</dependencyManagement>
```

Never import two generations together: their artifacts share their packages.

## Layout

All generations live on one branch and are released together, at this project's version; the generation is in each
artifactId, not in a branch or in the version.

```
pom.xml               the parent; one version property per member project
jinahyaee-bom-NN/     the bill of materials of Jakarta EE NN
```

## Members

| Project | Version | Generations | Note |
| --- | --- | --- | --- |
| [jinahya-persistence](https://github.com/jinahya/jinahya-persistence) | 0.7.1 | 11 | `-NN` artifacts from 0.7.0 |
| [jinahya-validation](https://github.com/jinahya/jinahya-validation) | 0.0.4 | 10, 11 | `-NN` artifacts from 0.0.3 |

Release a member before this project: a bill of materials is released only with released versions.

## Jakarta EE and Java SE

Each generation's minimum Java SE is the one its platform specification requires; a member compiles its main classes
for it, and nothing lower.

| Jakarta EE | Java SE, minimum | Also supported | Status |
| --- | --- | --- | --- |
| 9, 9.1 | 8 | 11 (9.1) | no bill of materials here |
| 10 | 11 | 17 | limited support upstream |
| 11 | 17 | 21 | current |
| 12 | 21 (planned) | 25 (planned) | not released; no `jakarta.jakartaee-bom` 12.x on Maven Central |

*See the [Jakarta EE Platform specifications](https://jakarta.ee/specifications/platform/).*

## Structuring a member

Every member (validation, persistence, and any other Jakarta area) follows the layout of
[jinahya-validation](https://github.com/jinahya/jinahya-validation), the reference implementation of these rules.

### Artifacts

* Each artifactId ends with its generation, `-NN` (`jinahya-validation-utils-11`), and the generations of one member
  share their packages; that is why a user must never mix two generations, and why each generation has its own bill of
  materials here.
* All generations live on one branch and are released together, at the member's version; never a branch or a version
  per generation.
* Every API of the area is `provided`: the platform, or the user's own implementation, supplies it.
* Each module has a `module-info.java` in `shared/<module>/src/main/java`, named after its package
  (`io.github.jinahya.validation.util`), never after its artifactId: the `-NN` suffix would derive an invalid
  automatic module name (`jinahya.validation.utils.11`), and every generation of a module shares its name, as it shares
  its packages.

### Layout

```
pom.xml                          plugins, and the versions not tied to a generation
shared/<module>/src/...          the sources every generation shares
<member>-NN/pom.xml              Jakarta EE NN: the platform BOM, the implementations, Java SE
  <member>-<module>-NN/          compiles shared/<module>/src/... plus its own src/, which holds only what differs
  README.md                      the generation's environment and version alignment
```

* Only the generation's parent, `<member>-NN/pom.xml`, sets what is tied to the generation: `jakarta.jakartaee-bom`
  (and through it every `jakarta.*` API), the minimum Java SE, and every implementation of a Jakarta specification,
  each on the newest release of the series aligned with that generation.
* The root `pom.xml` holds only what is the same for every generation (JUnit, Mockito, AssertJ, Lombok, SLF4J, the
  plugins).
* A source which a new generation's API breaks moves from `shared/` into the generations' own `src/`.

### Java versions

* **Main classes** are compiled for the generation's minimum Java SE (`maven.compiler.source`), and the enforcer
  requires a JDK at least that new.
* **Their runtime dependencies** (everything outside the `test` scope, `provided` included) must run on that minimum,
  too, enforced with `enforceBytecodeVersion`. A library shipped at `compile` scope may therefore need an older series
  in an older generation (e.g., AssertJ 3.27 in `jinahya-validation-test-utils-10`, since AssertJ 4 requires Java 17).
* **The build** uses one JDK for every generation: the latest LTS which can still compile for every generation's
  minimum (now JDK 25). Test classes are compiled for it (`maven.compiler.testSource`), and the test dependencies are on
  their latest releases which support it.

### Implementations

* Where a specification has several implementations (Hibernate Validator and Apache BVal; Hibernate ORM and
  EclipseLink), the tests run against one at a time, chosen with a profile of each generation's parent, one of them
  active by default; implementations are `test`-scoped.
* Tests specific to an implementation go in `src/test/java-<profile>`.
* A generation leaves an implementation, with its profile, out until it has a series for that generation.
* `./_mvn.sh` builds every generation against each implementation, and the GitHub Actions workflow runs one job per
  implementation, on the build JDK.

### Documentation

* The root `README.md` lists the artifacts per generation, and the generations with their specification version,
  minimum Java SE, and status, plus a survey of the next, unreleased generation.
* Each `<member>-NN/README.md` tables its environment, its alignment (the BOM's API versions against the
  implementations' versions in the build and the latest of their series), and its artifacts' runtime dependencies;
  update it whenever its `pom.xml` changes, and date each survey.

### Releasing

* Release the member, then update its version in this project's root `pom.xml`, its row in [Members](#members), and its
  artifacts in each `jinahyaee-bom-NN`, then release this project.
* Every release of every member counts against the `io.github.jinahya` namespace's monthly publishing limits on Maven
  Central (27 releases, 400 MB, 12,210 files, since support ticket #130571, 2026-11-13); batch changes rather than
  release each one.

## Adding a generation

1. Add a `jinahyaee-bom-NN` module, and list it in the root `pom.xml`.
2. List in it each member's artifacts of that generation.
3. Add a row to the tables above.
