# 🔹  Getting Started 
## Installation

```bash
pip install huggingface_hub webdataset h5py
```

## Downloading the Dataset

Shards may be retrieved from the Hugging Face Hub via the `huggingface_hub` library as demonstrated below.


**Stream directly from the Hub with authentication (recommended for large-scale training):**


```python
fs = HfFileSystem()
files = [fs.resolve_path(path) for path in fs.glob("hf://datasets/your-org/geomeld/data/*.tar")]
urls = [hf_hub_url(file.repo_id, file.path_in_repo, repo_type="dataset") for file in files]
urls = f"pipe: curl -s -L -H 'Authorization:Bearer {get_token()}' {'::'.join(urls)}"
ds = wds.WebDataset(urls).decode()
```

**Download a single shard:**

```python
shard_path = hf_hub_download(
    repo_id="your-org/geomeld",   # replace with actual repo ID
    filename="geomeld-00004_n.tar",
    repo_type="dataset",
    local_dir="./geomeld_shards"
)
```

**Download all shards (or filter by subset):**

```python
# Download the entire dataset
snapshot_download(
    repo_id="your-org/geomeld",   # replace with actual repo ID
    repo_type="dataset",
    local_dir="./geomeld_shards"
)
```

---

## 📖 Usage

Each `.tar` shard contains collection of `.h5` files. The following example demonstrates how to open a shard, deserialize the embedded HDF5 binary, and extract numerical arrays alongside associated metadata.

```python
TAR_PATH = "./geomeld_shards/geomeld-00004_n.tar"  # local path after download

dataset = wds.WebDataset(TAR_PATH)

for sample in dataset:
    key = sample["__key__"]

    # Each sample's HDF5 file is stored as raw bytes under the "h5" key
    h5_buffer = io.BytesIO(sample["h5"])

    with h5py.File(h5_buffer, "r") as f:

        # --- Metadata ---
        metadata_raw = f["metadata"][()]
        metadata_str = metadata_raw.decode("utf-8") if isinstance(metadata_raw, bytes) else str(metadata_raw)

        # --- Imagery arrays ---
        naip           = f["naip"][()]           # (3, 1280, 1280) uint16  — NAIP shards only
        sentinel2      = f["sentinel2"][()]       # (9 or 12, 128, 128) float32
        sentinel1      = f["sentinel1"][()]       # (8, 128, 128) float32
        aster          = f["aster"][()]           # (2, 128, 128) float32
        canopy_height  = f["canopy_height"][()]   # (2, 128, 128) float32

        # --- Segmentation masks ---
        esa_worldcover = f["esa_worldcover"][()]  # (1, 128, 128) uint8
        dynamic_world  = f["dynamic_world"][()]   # (1, 128, 128) uint8

    break  # remove to iterate over all samples
```

**Integration with a PyTorch DataLoader for model training:**

```python
def decode_sample(sample):
    """Decode a raw WebDataset sample containing .npy and .json files into tensors."""

    s2_buffer = io.BytesIO(sample["sentinel2.npy"])
    sentinel2 = torch.from_numpy(np.load(s2_buffer))

    s1_buffer = io.BytesIO(sample["sentinel1.npy"])
    sentinel1 = torch.from_numpy(np.load(s1_buffer))

    label_buffer = io.BytesIO(sample["esa_worldcover.npy"])
    esa_worldcover = torch.from_numpy(np.load(label_buffer))

    metadata = json.loads(sample["metadata.json"].decode("utf-8"))

    return {
        "sentinel2": sentinel2,
        "sentinel1": sentinel1,
        "label": esa_worldcover,
        "metadata": metadata,
    }


TAR_PATTERN = "https://huggingface.co/datasets/your-org/geomeld/resolve/main/data/geomeld-{00000..00002}_n.tar"

dataset = (
    wds.WebDataset(TAR_PATTERN, shardshuffle=100)
    .map(decode_sample)
    .batched(16)
)

loader = wds.WebLoader(dataset, num_workers=4)
```
