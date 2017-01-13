# OIDCAuthServer
##简介
本项目是一个实现IODC协议的认证服务。本身属于Spring Cloud为服务，依赖zookeeper、redis、mongodb，并且拓展实现了MongodbTokenStore、DefaultOAuth2ProviderTokenService等，可直接用于实现第三方登录。
支持OIDC（以及Oauth2）协议的各种模式。

## 各种URL以及返回数据示例（认证码模式）

**authorize request**

请求: GET  http://localhost:9090/auth/oauth/authorize?response_type=code&client_id=acme&redirect_uri=http://example.com

回复: 302  http://example.com?code=883jhy

**token request**

curl -X POST -H "Content-Type: application/x-www-form-urlencoded" -d 'grant_type=authorization_code&code=d4IYR5&redirect_uri=http://example.com&client_id=acme&client_secret=acmesecret' "http://127.0.0.1:9090/auth/oidc/token"

{
  "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9.eyJzdWIiOiJhZG1pbiIsImF6cCI6ImFjbWUiLCJpc3MiOiJodHRwOlwvXC93d3d3LndhbmcuY29tIiwiZXhwIjoxNDg0MzYwNTI0LCJpYXQiOjE0ODQyNzQxMjQsImp0aSI6IjI3ZmU5NjlmLTM1YTEtNDljOS1hOWQ0LTMzYTM2MWRiZWRlNyJ9.suW91W1xKVmLXxr68uhM_j8Ycd3OF1eWBRSQpU2HYFxmfrn0uSQSB5PwOI3Ezr90qlvuX79onwKbHH7B89eoT68dY8ubfQNJr0wjeYUGmT1aweLgLDxQWpcqOomgEPm7QxoUMmtvBQIeYmEi86FZw9BEUzOLIqGC3nPfowgIy4yhrsBjTvfOjZrQENcKf8CNHXlkWv5lkEYOd-NYdQSfEy1CcmijXyTG47WVerct38VRgM5Mwqy1d3Nx0rt_7nGvOTuz7rbmLHkMsZUBwNc89h_UID-BfBckt006WEc1URPXZt1wJGmK7Np9q3AQ5irxsswsatGBVKh8tUnz8MLuvA",
  "token_type": "Bearer",
  "refresh_token": "eyJhbGciOiJub25lIn0.eyJleHAiOjE0ODQ4Nzg5MjQsImp0aSI6ImE1YzEyMGY3LWM3NTEtNDcwMC1hNjNkLWQ0MzBkMzlhNTZhYyJ9.",
  "expires_in": 86399,
  "scope": "read openid",
  "id_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9.eyJzdWIiOiJhZG1pbiIsImF1ZCI6ImFjbWUiLCJ1c2VyX25hbWUiOiJhZG1pbiIsImlzcyI6Imh0dHA6XC9cL3d3d3cud2FuZy5jb20iLCJleHAiOjE0ODQzNjA1MjQsImlhdCI6MTQ4NDI3NDEyNCwianRpIjoiMDYzNjczMzYtYTdiNi00NmIxLWIzMTYtNjdhYTdhNjVjM2FmIn0.LdCWUxk_rK4x6eLQf-GCEhlcwOixBq2VqYNLgQ3eRWiG0iiDLoToEQ9vAdiZ7ge3ue4P5NVv90q4exoBnN5KpUk5tVRESrFI6OC5uNAVYLj6oHZU1Ka7b9tT06KwZqiyXGYRM_s0img6w12Lk1E7MKhwHeKTxW1Q2hiHTwWAVdETEPDX-iIl5Et9yeGhRatKtgglgPYvecGKT8zvhnPRC1oFCqKGHwfrdLPcJCg-LQLxmGf0rLjQPbYxJmJsix2aRrxpb-uTZQ5gTkDISnlEzWyXEvLPx0MxQ6hn2lQ6LAVC-tQbw7TmMrbfGONv7b6vGbnkyUghx9WS1czLPaT1AQ"
}

**refresh token request**

curl -X POST -H "Content-Type: application/x-www-form-urlencoded" -d 'grant_type=refresh_token&refresh_token=eyJhbGciOiJub25lIn0.eyJleHAiOjE0ODQ4Nzg5MjQsImp0aSI6ImE1YzEyMGY3LWM3NTEtNDcwMC1hNjNkLWQ0MzBkMzlhNTZhYyJ9.&client_id=acme&client_secret=acmesecret' "http://127.0.0.1:9090/auth/oidc/token"

{
  "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9.eyJzdWIiOiJhZG1pbiIsImF6cCI6ImFjbWUiLCJpc3MiOiJodHRwOlwvXC93d3d3LndhbmcuY29tIiwiaWF0IjoxNDg0MjkwMzIyLCJqdGkiOiI0ZTk3MDgxOS1hNTIyLTQxZDctYTYwNy1iZjhlYWExYjU1ZjEifQ.BTNqLohyMpIsTctb6G1qxTebJuKgJwpxIYgbKcfL-EF-DukypzLLKQHFOhxaklIGLCZeJsuJ8_51i8DlGqHf3FDZqVqpDfG9wVehmxKKElwGUImD5ySmx0cIs2pkwjbx8cFknE-SRpYxpP5COFjS9SWgK1G4PeRK2s596CScPTSVgHfZ8uSEGrlkwlVrUmnP2II78UfFhrIV2pstZEKwmtqJG7AO8JnK6e84DRZv6ehpRQ3pGU1rrN2S7hOZZHaxlHL-9MOfDvpOfsnCJlPTC9yYVmUcxTIqqk_E_Lw6DVj5JsX_9IegcVp-QpMwYdZoDgYFbuvOg53usL_ilTyWYg",
  "token_type": "Bearer",
  "refresh_token": "eyJhbGciOiJub25lIn0.eyJleHAiOjE0ODQ4Nzg5MjQsImp0aSI6ImE1YzEyMGY3LWM3NTEtNDcwMC1hNjNkLWQ0MzBkMzlhNTZhYyJ9.",
  "id_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9.eyJzdWIiOiJhZG1pbiIsImF1ZCI6ImFjbWUiLCJ1c2VyX25hbWUiOiJhZG1pbiIsImlzcyI6Imh0dHA6XC9cL3d3d3cud2FuZy5jb20iLCJleHAiOjE0ODQzNzY3MjMsImlhdCI6MTQ4NDI5MDMyMiwianRpIjoiODk5OWIwOTQtNjY5ZC00YTUzLTk1YjktZTYzYmZjYmQ5ZjAyIn0.hPIARvL89N_xFsY37Gp45yqOZRrCKENmQUMTWUYVhyEWWnhkfKX19jpIy-Taq3cylX61wZCXLGalqwdKSBqzH3XYd8OkGPS3pkGnaincMtWa-kY3TmYfCRdVsp2mOiLRda5T5kP4N8Nokwg-okZ8YKHVb1IReAMirhfkVRYgB_qo3ZBD5xF0Dwj28lWFNYc1JT8heB7JlqDdwiD55b967-GmNlNEw-6wkdfvH9An97u2qo-6Z2wVvUVUPHLZBW94L9ez-Imt3cB9x4gQ5AnS_N6oRnw-poKjR0O2lPRBzeM1F5jdzhTFt8K9D4X7l__VCOY4luOwLQTzgIaRn5gj4Q"
}

