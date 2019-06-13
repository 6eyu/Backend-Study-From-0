# Servlet Filter 

```
@Component
@Order(1)
public class AccountFilter implements Filter {

    @Autowired
    Gson gson;

    @Autowired
    AccountRepository accountRepository;

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {

    }

    @Transactional(readOnly = true) //注意开启事物,否则TAccount中的Object无法从数据库中读出来. 因为JPA机制 是随用随读 增加效率.
    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {

        HttpServletRequest httpRequest = (HttpServletRequest)servletRequest;
        HttpServletResponse httpResponse = (HttpServletResponse)servletResponse;

        RequestHeaderDTO requestHeaderDto = gson.fromJson(httpRequest.getHeader("sessiondata"), RequestHeaderDto.class);

        TAccount tAccount = accountRepository.findByAccountId(requestHeaderDto.getUser().getAccountId())
                .orElseThrow(() -> new AccountException(" Account Id " + requestHeaderDto.getUser().getAccountId() + " does not exist."));

        httpRequest.setAttribute("accountdata", tAccount);

        filterChain.doFilter(httpRequest, httpResponse);
    }

    @Override
    public void destroy() {

    }
}

```



