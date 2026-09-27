# Data

No raw images or labels are committed to this repository, and that is
intentional (see the root README's Reproducibility Checklist).

The frozen dataset this project's results depend on is exported from
Roboflow once and published as a **GitHub Release asset** (a zip file),
fingerprinted with a SHA256 checksum. Notebooks download it at run time
by URL, verify the checksum, and extract it — no Roboflow account or API
key required to reproduce results.

This folder is reserved for small, non-binary reference files only
(e.g., a copy of `data.yaml`, a class list). It does not and will not
contain the dataset itself.

Not added yet — coming in a later step, once the dataset is exported and
released.
