FROM jupyter/pyspark-notebook:python-3.11

# (opcional) libs extras
COPY docker/requirements.txt /tmp/requirements.txt
RUN pip install --no-cache-dir -r /tmp/requirements.txt

# Ajustes úteis
ENV PYSPARK_PYTHON=python
ENV PYSPARK_DRIVER_PYTHON=python
