# jinahyaee-bom

Bills of materials of the `jinahya-*` artifacts, one per Jakarta EE generation, as
`jakarta.platform:jakarta.jakartaee-bom` is for the `jakarta.*` APIs.

| Jakarta EE | Bill of materials | Manages |
| --- | --- | --- |
| 10 | `io.github.jinahya:jinahyaee-bom-10` | `jinahya-validation-*-10` |
| 11 | `io.github.jinahya:jinahyaee-bom-11` | `jinahya-persistence-*`, `jinahya-validation-*-11` |

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
| [jinahya-persistence](https://github.com/jinahya/jinahya-persistence) | 0.6.9 | 11 | no generation in its artifactIds |
| [jinahya-validation](https://github.com/jinahya/jinahya-validation) | 0.0.3-SNAPSHOT | 10, 11 | not released yet; `-NN` artifacts from 0.0.3 |

Release a member before this project: a bill of materials is released only with released versions.

## Adding a generation

1. Add a `jinahyaee-bom-NN` module, and list it in the root `pom.xml`.
2. List in it each member's artifacts of that generation.
3. Add a row to the tables above.
