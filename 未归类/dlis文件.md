```python
import dlisio

with dlisio.dlis.load("./data/image_from_cosl/太古界3井.dlis") as files:

    print(files.describe())

    lf = files[0]  # 因为只有一个 Logical File

    for ch in lf.channels:

        print(ch.name)

    for ch in lf.channels:

        print(ch.curves())
```

```txt
------------- Physical File ------------- Number of Logical Files : 1 Description : LogicalFile(FILE_ID) Frames : 1 Channels : 7 UDIMG0 UDIMG120 UDIMG150 UDIMG30 UDIMG60 UDIMG90 TDEP [[0. 0. 0. ... 0. 0. 0.] [0. 0. 0. ... 0. 0. 0.] [0. 0. 0. ... 0. 0. 0.] ... [0. 0. 0. ... 0. 0. 0.] [0. 0. 0. ... 0. 0. 0.] [0. 0. 0. ... 0. 0. 0.]] [[0. 0. 0. ... 0. 0. 0.]

...

3166.6052 3166.7576 3166.91 3167.0625 3167.2148 3167.3672 3167.5195 3167.672 3167.8245 3167.9768 3168.1292 3168.2815 3168.434 3168.5864 3168.7388 3168.891 3169.0437 3169.196 3169.3484 3169.5007 3169.6533 3169.8057 3169.958 ]
```