# Реалізація інформаційного та програмного забезпечення
```
USE my_database;

SET FOREIGN_KEY_CHECKS = 0;

DROP TABLE IF EXISTS Access;
DROP TABLE IF EXISTS DatarecordCategory;
DROP TABLE IF EXISTS DatarecordTag;
DROP TABLE IF EXISTS User;
DROP TABLE IF EXISTS Role;
DROP TABLE IF EXISTS Datarecord;
DROP TABLE IF EXISTS Category;
DROP TABLE IF EXISTS Tag;

SET FOREIGN_KEY_CHECKS = 1;

CREATE TABLE Role (
    id CHAR(36) PRIMARY KEY,
    name TEXT NOT NULL
);

CREATE TABLE User (
    id CHAR(36) PRIMARY KEY,
    name TEXT NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    password TEXT NOT NULL,
    roleId CHAR(36),
    FOREIGN KEY (roleId) REFERENCES Role(id) ON DELETE SET NULL
);

CREATE TABLE Datarecord (
    id CHAR(36) PRIMARY KEY,
    name TEXT NOT NULL,
    data TEXT NOT NULL,
    type TEXT NOT NULL,
    time TIMESTAMP NOT NULL,
    description TEXT
);

CREATE TABLE  Access (
    id CHAR(36) PRIMARY KEY,
    userId CHAR(36),
    datarecordId CHAR(36),
    time TIMESTAMP NOT NULL,
    type TEXT NOT NULL,
    FOREIGN KEY (userId) REFERENCES User(id) ON DELETE CASCADE,
    FOREIGN KEY (datarecordId) REFERENCES Datarecord(id) ON DELETE CASCADE
);

CREATE TABLE  Tag (
    id CHAR(36) PRIMARY KEY,
    name TEXT NOT NULL
);

CREATE TABLE Category (
    id CHAR(36) PRIMARY KEY,
    name TEXT NOT NULL,
    parentCategoryId CHAR(36),
    FOREIGN KEY (parentCategoryId) REFERENCES Category(id) ON DELETE SET NULL
);

CREATE TABLE DatarecordTag (
    id CHAR(36) PRIMARY KEY,
    datarecordId CHAR(36),
    tagId CHAR(36),
    FOREIGN KEY (datarecordId) REFERENCES Datarecord(id) ON DELETE CASCADE,
    FOREIGN KEY (tagId) REFERENCES Tag(id) ON DELETE CASCADE
);

CREATE TABLE  DatarecordCategory (
    id CHAR(36) PRIMARY KEY,
    datarecordId CHAR(36),
    categoryId CHAR(36),
    FOREIGN KEY (datarecordId) REFERENCES Datarecord(id) ON DELETE CASCADE,
    FOREIGN KEY (categoryId) REFERENCES Category(id) ON DELETE CASCADE
);
```
## RESTfull сервіс для управління даними

# app.py
```
from flask import Flask, request, jsonify
from datetime import datetime

app = Flask(__name__)

# Список для зберігання записів (замінник бази даних)
access_records = []


# Створення нового запису
@app.route('/access', methods=['POST'])
def create_access():
    data = request.get_json()
    if not data or not all(key in data for key in ['userId', 'datarecordId', 'time', 'type']):
        return jsonify({'error': 'Invalid input data'}), 400

    new_id = len(access_records) + 1
    new_record = {
        'id': new_id,
        'userId': data['userId'],
        'datarecordId': data['datarecordId'],
        'time': data['time'],
        'type': data['type']
    }
    access_records.append(new_record)
    return jsonify(new_record), 201


# Читання записів
@app.route('/access', methods=['GET'])
def read_access():
    userId = request.args.get('userId')
    datarecordId = request.args.get('datarecordId')
    record_type = request.args.get('type')

    filtered_records = access_records

    if userId:
        filtered_records = [r for r in filtered_records if r['userId'] == int(userId)]
    if datarecordId:
        filtered_records = [r for r in filtered_records if r['datarecordId'] == int(datarecordId)]
    if record_type:
        filtered_records = [r for r in filtered_records if r['type'] == record_type]

    return jsonify(filtered_records)


# Оновлення запису
@app.route('/access/<int:id>', methods=['PUT'])
def update_access(id):
    data = request.get_json()
    for record in access_records:
        if record['id'] == id:
            record.update(data)
            return jsonify(record)
    return jsonify({'message': 'Record not found'}), 404


# Видалення запису
@app.route('/access/<int:id>', methods=['DELETE'])
def delete_access(id):
    global access_records
    access_records = [r for r in access_records if r['id'] != id]
    return '', 204


# Отримання записів за часом
@app.route('/access/time', methods=['GET'])
def access_by_time():
    start_time = datetime.fromisoformat(request.args.get('startTime'))
    end_time = datetime.fromisoformat(request.args.get('endTime'))

    filtered_records = [r for r in access_records if start_time <= datetime.fromisoformat(r['time']) <= end_time]
    return jsonify(filtered_records)


if __name__ == '__main__':
    app.run(debug=True)
```
