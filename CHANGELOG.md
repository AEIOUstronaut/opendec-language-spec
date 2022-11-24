# Language Specification Versioning
The OpenDec language specification consists of three parts: the major version,
the minor version, and the revision. Each part is separated by a period. When
changes to the specification are made, then follow these rules to understand
which part of the version number should be incremented:
* **Major**:    Increment when adding or removing a feature
* **Minor**:    Increment when modifying a feature (e.g. changing command names,
                    changing parameter types)
* **Revision**: Increment when there are non-functional changes (e.g. grammar,
                    word choice) to correct or improve clarity of the document.

The minor version and revision cannot exceed 20. If the revision is 20 and needs
to be incremented, then the minor version should be incremented. Similarly, if
the minor version is 20 and needs to be incremented, then the major version
should be incremented. See the examples below.
```
1.0.20  -> 1.1.0    // Increment revision
1.1.20  -> 1.2.0    // Increment revision
1.20.0  -> 2.0.0    // Increment minor version
1.20.5  -> 2.0.0    // Increment minor version
1.20.20 -> 2.0.0    // Increment minor version or revision
```

Small changes such as fixing typos and making minor corrections (e.g. swapping
two words in the wrong order) do not require a major, minor, or revision update.

New changelog entries must follow the markdown template below. Newer entries
should appear higher up in this document than older entries.
```
**v<Major>.<Minor>.<Revision>** (YYYY-MM-DD)
* Change 1
* Change 2
* ...
* Change N
```


# Changelog
**v1.0.0** (2022-11-23)
* Initial document created
