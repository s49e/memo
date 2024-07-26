``` java

@Controller
@Slf4j
@RequiredArgsConstructor
@RequestMapping("/admin/product")
public class AdminProductController {

	private static final int DEFAULT_START_PAGE = 1;
	private static final int DEFAULT_PAGE_SIZE = 5;

	private final ProductClient productClient;

	@ProductJwtValidate
	@PostMapping("/edit/{id}")
	public String editProduct(@PathVariable("id") int id, @ModelAttribute ProductUpdateForm product) {
		productClient.updateProduct(id, ProductUpdateForm.convertFormToReq(product));
		return "redirect:/admin/product?query=" + URLEncoder.encode(product.getName(), StandardCharsets.UTF_8);

	}


```
``` java

@ActiveProfiles("test")
@WebMvcTest(value = AdminProductController.class)
class AdminProductControllerTest {

	@Autowired
	private MockMvc mockMvc;

	@MockBean
	private ProductClient productClient;
	@MockBean
	private ProductTagClient productTagClient;
	@MockBean
	private TagClient tagClient;
	@MockBean
	private JwtService jwtService;

	@MockBean
	private CookieUtils cookieUtils;
	@Autowired
	private WebApplicationContext webApplicationContext;

	@MockBean
	private CartInterceptor cartInterceptor;

	@BeforeEach
	void setUp() {
		MockitoAnnotations.openMocks(this);
		mockMvc = MockMvcBuilders.webAppContextSetup(webApplicationContext)
			.apply(SecurityMockMvcConfigurers.springSecurity())
			.build();

		Cookie mockCookie = new Cookie(COOKIE_JWT_ACCESS_KEY, "valid-token");
		when(cookieUtils.getCookie(any(HttpServletRequest.class), anyString())).thenReturn(Optional.of(mockCookie));
		when(jwtService.getInfoMapFromJwt(any(HttpServletRequest.class), any(HttpServletResponse.class)))
			.thenReturn(Map.of(
				JwtService.USER_ID, 1,
				JwtService.LOGIN_ID, "test",
				JwtService.ROLE, "ROLE_ADMIN"
			));
	}

	@Test
	@WithMockUser(roles = "ADMIN")
	void testEditProduct() throws Exception {

		int productId = 1;
		ProductUpdateForm mockProduct = new ProductUpdateForm(); // Assuming Product class exists
		mockProduct.setId(productId);
		mockProduct.setName("Test Product");

		when(productClient.updateProduct(anyInt(), any(ProductUpdateRequest.class))).thenReturn(new ProductResponse());

		mockMvc.perform(MockMvcRequestBuilders.post("/admin/product/edit/{id}", productId)
				.with(csrf())
				.flashAttr("productUpdateForm", mockProduct))
			// 이렇게 값을 넘겨주려면 controller에서 @ModelAttribute("product")이렇게 해야함
			// @ModelAttribute ProductUpdateForm product 이렇게만 하면 param으로 넘겨준 값을 알아서 productUpdateForm에 맵핑해줌
            // = 테스트에서는 제대로 맵핑 안해줌


			.andExpect(status().is3xxRedirection());
	}
}

```