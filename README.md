public class CustomClientException extends RuntimeException {
    private final HttpStatus status;

    public CustomClientException(String message, HttpStatus status) {
        super(message);
        this.status = status;
    }

    public HttpStatus getStatus() {
        return status;
    }
}

public class CustomServerException extends RuntimeException {
    private final HttpStatus status;

    public CustomServerException(String message, HttpStatus status) {
        super(message);
        this.status = status;
    }

    public HttpStatus getStatus() {
        return status;
    }
}

public class CustomGenericException extends RuntimeException {
    private final HttpStatus status;

    public CustomGenericException(String message, HttpStatus status) {
        super(message);
        this.status = status;
    }

    public HttpStatus getStatus() {
        return status;
    }
}
