# ng-align 
官网:  https://ng-alain.com/zh

github: https://github.com/ng-alain/ng-alain/

# token刷新机制
1. ng-alain自带的token刷新机制：过期时自动被@delon/auth 拦截 返回401 ,然后尝试refreshtoken,
    但是有问题：由于原本的token已过期,在refreshtoken时,会被强制跳转到登录页。
2. 由于1中的问题,需要做些调整。首先在global-config.module.ts中配置token_exp_offset(秒),可参考： https://ng-alain.com/auth/getting-started/zh#AlainAuthConfig 当此值为负数,表示token过期指定秒数内,也可被ng-alain带着传向后端。配置-t,即延长t秒。(t为token有效期)  
3. 前端登录后返回有效期为t的token 和 有效期为2t的refresh_token
4. 后端拦截到前端传递的过期的token后,并发现token的过期时间在t内,可刷新,然后返回401,前端带着 refresh_token 尝试刷新,即可。

# 其他token刷新机制
以上方法,是因为使用ng-alain的问题,所以比较繁琐。若自己做整个过程也可使用其他方式。
1. 后端拦截到token后,发现token快要过期了,可在HttpServletResponse中返回一个refresh_token ,前端发现有refresh_token,则把refresh_token赋值给token即可。
2. 后端生成的token存一份2倍有效期的到redis中,可以以PC/mobile + 用户ID 若要支持不限制登录终端可使用 MAC+用户ID 为key,后端拦截到前端过期的token,若能在redis中获取到,说明可刷新,需返回新的token给前端,并把redis中的原有值覆盖,此次请求也是有效的。
3. 后端在验证成功后,返回有效期为t的token和有效期为2t的refresh_token。前端在发现token过期后,尝试使用refresh_token获取新的token。若refresh_token也过期了,则提示过期跳转登录页。(参照: https://www.jianshu.com/p/5a60b757af34)
