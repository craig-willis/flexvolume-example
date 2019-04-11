FROM python:3.6

RUN pip install wsgidav cheroot

RUN mkdir /davroot

CMD ["wsgidav", "--host=0.0.0.0", "--port=8080", "--root=/davroot", "--auth=anonymous"]
