# Sử dụng AsyncStorage để làm trang đăng ký, đăng nhập

## Mục tiêu

- Biết cách sự dụng AsyncStorage để lưu dữ liệu vào local storage.
- Biết cách sử dụng dữ liệu có cấu trúc (JSON) để quản lý dữ liệu trong Storage.

## Nội dung

- Tạo một ứng dụng cho phép người dùng đăng nhập, đăng ký.
- Ứng dụng cho phép lưu lại những người dùng đã đăng ký với `username` và `password`.

## Hướng dẫn

- Project sẽ sử dụng `react-navigation` để tạo các tabs trên giao diện và sử dụng `md5` package để mã hóa `password`.

Thêm các packages vào project:

```
npm i --save react-navigation md5
```

- Tạo 1 file `StorageServices.js` để tương tác `AysncStorage` với vai trò là 1 services cung cấp các tác vụ `createNewUser` và `getUserByUserName`:

```javascript
import {AsyncStorage} from 'react-native'

const _getKey = (key) => `cg@users:${key}`

export const getUserByUserName = username => {
    if (!username) {
        return Promise.reject(new Error('Username is empty'))
    }

    const key = _getKey(username)

    return AsyncStorage.getItem(key)
        .then(str => {
            if (!str) {
                throw new Error('User not found')
            }

            try {
                return JSON.parse(str)
            } catch (error) {
                //Remove data is broken
                AsyncStorage.removeItem(key)

                throw error
            }
        })
        .catch(error => {
            return Promise.resolve(false)
        })
}

export const createNewUser = ({username, name, password}) => {
    return getUserByUserName(username)
        .then(user => {
            if (user) {
                throw new Error('The user already exists')
            }

            const newUser = {username, name, password}
            const key = _getKey(username)

            return AsyncStorage.setItem(key, JSON.stringify(newUser))
        })
}
```

- Tạo 1 file `AuthServices.js` để xử lý các tác vụ đăng nhập/đăng ký bằng cách sử dụng `StorageServices`:

```javascript
import {createNewUser, getUserByUserName} from "./StorageServices"
import md5 from 'md5'

const _hashPassword = password => {
    return Promise.resolve(md5(password))
}

const _verifyPassword = (password, hash) => {
    const hashed = md5(password)

    return Promise.resolve(hashed === hash)
}

export const register = ({username, password, name}) => {
    if (!password || password.length < 6) {
        return Promise.reject(new Error('Your password is too short'))
    }

    return _hashPassword(password)
        .then(hash => {
            const user = {username, password: hash, name}

            return createNewUser(user)
        })
}

export const login = ({username, password}) => {
    if (!username) {
        return Promise.reject(new Error('Please enter your username!'))
    }

    return getUserByUserName(username)
        .then(user => {
            if (!user) {
                throw new Error('User is not exits')
            }

            const passwordHashed = user.password || '';

            return _verifyPassword(password, passwordHashed)
                .then(success => {
                    if (!success) {
                        throw new Error('Your password is wrong')
                    }

                    delete user.password
                    _loginSuccess(user)

                    return user
                })
        })
}

export const logout = () => {
    _store.isAuthenticated = false;
    _store.user = {}

    _broadcast()
}

const _store = {
    isAuthenticated: false,
    user: {},
    subscribers: []
}

const _loginSuccess = (user) => {
    _store.user = {
        ...user
    }
    _store.isAuthenticated = true
    _broadcast()
}

const _broadcast = () => {
    _store.subscribers.forEach(subscriber => {
        typeof subscriber === 'function' && subscriber(_store.user)
    })
}

export const isAuthenticated = () => _store.isAuthenticated

export const getCurrentUser = () => _store.user

export const subscribeAuthentication = subscriber => {
    if (typeof subscriber !== 'function') return
    if (_store.subscribers.indexOf(subscriber) !== -1) return

    _store.subscribers = [].concat(_store.subscribers, [subscriber])
}

export const unsubscribeAuthentication = subscriber => {
    _store.subscribers = _store.subscribers.filter(_sub => _sub !== subscriber)
}
```

- Sử dụng `react-navigation` để tạo 2 tabs là `HomePage` và `LoginPage`. Ở trong 2 screens sẽ sử dụng các functions do `AuthServices` xử lý các tác vụ đăng nhập và đăng ký.

## Mã nguồn

Tham khảo tại: https://github.com/tutv/rn-login

## Ảnh demo

![Login](/demo/login.jpeg)

![Login failed](/demo/login-failed.jpeg)

![Home page](/demo/home.jpeg)