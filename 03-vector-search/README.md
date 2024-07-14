# Homework #3 for LLM Zoomcamp (by DataTalksClub)

### Preparing the environment
Install required python packages

```bash
pip install requests jupyter elasticsearch tqdm sentence-transformers pandas
```

### For questions Q5 and Q6

Run `elasticsearch` container:

```bash
docker run -it \
    --rm \
    --name elasticsearch \
    -p 9200:9200 \
    -p 9300:9300 \
    -e "discovery.type=single-node" \
    -e "xpack.security.enabled=false" \
    docker.elastic.co/elasticsearch/elasticsearch:8.4.3
```