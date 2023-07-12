# Content

[Read the doc](https://correctexam.readthedocs.io/) repository for [correctExam](https://correctexam.github.io/) project. 

# To run it locally

```bash
git clone https://github.com/correctexam/documentation-french/
cd documentation/docs
pip install -r ./requirements.txt
make html
cd build/html
python3 -m http.server 8084
```

