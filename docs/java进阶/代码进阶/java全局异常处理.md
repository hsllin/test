通常一个web程序在运行过程中，由于用户的操作不当，或者程序的bug，有大量需要处理的异常。其中有些异常是需要暴露给用户的，比如登陆超时，权限不足等等。可以通过弹出提示信息的方式告诉用户出了什么错误。

而这就表示在程序中需要一个机制，去处理这些异常，将程序的异常转换为用户可读的异常。而且最重要的，是要将这个机制统一，提供统一的异常处理。因为我设计这个结构的主要目的是为了简化代码。

实现原理：利用@RestControllerAdvice注解，有这个注解的类中的方法的某些注解会应用到所有的Controller里，其中就包括@ExceptionHandler注解。

```java
@RestControllerAdvice  
@Slf4j  
@RequiredArgsConstructor  
public class ExceptionHandlers {  
private final MessageSourceHandler messageSourceHandler;  
  
public static String ERROR = "错误 ";  
public static String SQL_ERROR = "sql异常 ";  
public static String HTTP_ERROR = "http异常 ";  
public static String PARAMS_ERROR = "参数校验错误 ";  
public static String ILLEGAL_PARAMS_ERROR = "非法参数异常 ";  
  
private String appendMsg(String key, Throwable throwable) {  
return key + ": " + throwable;  
}  
  
private String appendMsg(String key, String msg) {  
return key + ": " + msg;  
}  
  
@ExceptionHandler(Exception.class)  
public Response<Object> exceptionHandler(Exception e) {  
log.error(ERROR, e);  
return Response.failure(e.getMessage(), appendMsg(ERROR, ExceptionUtils.rootCauseToString(e)), null);  
}  
  
@ExceptionHandler(SQLException.class)  
public Response<Object> sqlExceptionHandler(Exception e) {  
log.error(SQL_ERROR, e);  
return Response.failure(e.getMessage(), appendMsg(SQL_ERROR, ExceptionUtils.rootCauseToString(e)), null);  
}  
  
  
/**  
* http参数异常  
*  
* @param e  
* @return  
*/  
@ExceptionHandler({HttpMessageNotReadableException.class, HttpRequestMethodNotSupportedException.class, MethodArgumentTypeMismatchException.class})  
public Response<Object> httpMessageNotReadableExceptionHandler(Exception e) {  
log.error(HTTP_ERROR, e);  
return Response.failure(e.getMessage(), appendMsg(HTTP_ERROR, ExceptionUtils.rootCauseToString(e)), null);  
}  
  
/**  
* get参数验证异常 valid异常  
*  
* @param e  
* @return  
*/  
@ExceptionHandler({BindException.class})  
public Response<Object> validateExceptionHandler(BindException e) {  
String defaultMessage = e.getBindingResult().getFieldErrors().get(0).getDefaultMessage();  
FailedMessage msg = getMessage(defaultMessage, null);  
log.error(buildErrorMsg(msg.getMessage(), e.getMessage()), e);  
return getFailedResponse(msg, PARAMS_ERROR, e);  
}  
  
/**  
* post参数验证异常 valid异常  
*  
* @param e  
* @return  
*/  
@ExceptionHandler({MethodArgumentNotValidException.class})  
public Response<Object> validateExceptionHandler(MethodArgumentNotValidException e) {  
String defaultMessage = e.getBindingResult().getFieldErrors().get(0).getDefaultMessage();  
FailedMessage msg = getMessage(defaultMessage, null);  
log.error(buildErrorMsg(msg.getMessage(), e.getMessage()), e);  
return getFailedResponse(msg, PARAMS_ERROR, e);  
}  
  
  
/**  
* 自定义校验异常  
*/  
@ExceptionHandler(AssertFailExceptionBase.class)  
public Response<Object> assertExceptionHandler(AssertFailExceptionBase e) {  
FailedMessage msg = getMessage(e.getMessage(), e.getMsgArgs());  
log.error(buildErrorMsg(msg, e), e);  
return getFailedResponse(msg, e.getErrMessage(), e);  
}  
  
/**  
* 认证失败异常  
*/  
@ExceptionHandler(AuthFailedException.class)  
public ResponseEntity<String> authFailedExceptionHandler(AuthFailedException e) {  
FailedMessage msg = getMessage(e.getMessage(), e.getMsgArgs());  
return ResponseEntity.status(HttpStatus.UNAUTHORIZED).body(msg.getMessage());  
}  
  
/**  
* 自定义异常  
*  
* @param e  
* @return  
*/  
@ExceptionHandler(BaseServerException.class)  
public Response<Object> baseServerExceptionHandler(BaseServerException e) {  
FailedMessage msg = getMessage(e.getMessage(), e.getMsgArgs());  
log.error(buildErrorMsg(msg, e), e);  
return getFailedResponse(msg, e.getErrMessage(), e);  
}  
  
  
/**  
* 非法参数异常  
*  
* @param e  
* @return  
*/  
@ExceptionHandler(IllegalArgumentException.class)  
public Response<Object> baseServerExceptionHandler(IllegalArgumentException e) {  
FailedMessage msg = getMessage(e.getMessage(), null);  
log.error(buildErrorMsg(msg.getMessage(), e.getMessage()), e);  
return getFailedResponse(msg, PARAMS_ERROR, e);  
}  
  
  
private Response<Object> getFailedResponse(FailedMessage msg, String errMsg, Exception ex) {  
return new Response<>(  
msg.getCode(),  
msg.getMessage(),  
appendMsg(StringUtils.defaultString(errMsg), ExceptionUtils.rootCauseToString(ex)), null);  
}  
  
private FailedMessage getMessage(String msg, Object[] msgArgs) {  
return new FailedMessage(messageSourceHandler.getMessage(msg, msgArgs, msg));  
}  
  
private String buildErrorMsg(String originMsg, String readableMsg) {  
return (Objects.equals(originMsg, readableMsg) ?  
originMsg : readableMsg + "(" + originMsg + ")");  
}  
  
private String buildErrorMsg(FailedMessage msg, BaseServerException e) {  
StringBuilder sb = new StringBuilder();  
if (Objects.equals(e.getMessage(), msg.getMessage())) {  
sb.append(e.getMessage());  
} else {  
sb.append(msg.getMessage()).append("(").append(e.getMessage()).append(")");  
}  
if (e.getErrMessage() != null) {  
sb.append("=>").append(e.getErrMessage());  
}  
return sb.toString();  
}  
  
@Getter  
@Setter  
public static class FailedMessage {  
private Integer code = Response.CODE_FAILURE;  
private String message;  
  
public FailedMessage(String fullMsg) {  
try {  
String[] split = fullMsg.split(":::", 2);  
if (split.length < 2) {  
this.message = fullMsg;  
} else {  
this.code = Integer.parseInt(split[0]);  
this.message = split[1];  
}  
} catch (Exception ex) {  
this.code = Response.CODE_FAILURE;  
this.message = fullMsg;  
}  
}  
}  
}

```
