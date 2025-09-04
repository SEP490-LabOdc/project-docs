# Contract Service - Microservice cho quản lý hợp đồng

## 1. Tổng quan

**Chức năng chính**: Quản lý hợp đồng, tạo PDF và xử lý chữ ký số

**Công nghệ được sử dụng**:
- Spring Boot: Framework chính để phát triển microservice
- JPA/Hibernate: ORM để tương tác với database
- PostgreSQL: Lưu trữ thông tin hợp đồng
- iText/PDFBox: Thư viện xử lý PDF và chữ ký số
- Spring Cloud: Tích hợp với các microservice khác
- Kafka: Xử lý sự kiện và thông báo

## 2. Cấu trúc dự án

```
contract-service/
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/odc/contract/
│   │   │       ├── config/
│   │   │       │   ├── KafkaConfig.java
│   │   │       │   ├── SecurityConfig.java
│   │   │       │   └── PdfConfig.java
│   │   │       ├── controller/
│   │   │       │   └── ContractController.java
│   │   │       ├── dto/
│   │   │       │   ├── ContractDTO.java
│   │   │       │   ├── ContractRequestDTO.java
│   │   │       │   └── ContractResponseDTO.java
│   │   │       ├── exception/
│   │   │       │   ├── ContractNotFoundException.java
│   │   │       │   ├── PdfGenerationException.java
│   │   │       │   └── SignatureException.java
│   │   │       ├── model/
│   │   │       │   └── Contract.java
│   │   │       ├── repository/
│   │   │       │   └── ContractRepository.java
│   │   │       ├── service/
│   │   │       │   ├── ContractService.java
│   │   │       │   ├── ContractServiceImpl.java
│   │   │       │   ├── PdfService.java
│   │   │       │   ├── PdfServiceImpl.java
│   │   │       │   ├── SignatureService.java
│   │   │       │   └── SignatureServiceImpl.java
│   │   │       ├── client/
│   │   │       │   ├── BusinessServiceClient.java
│   │   │       │   └── UserServiceClient.java
│   │   │       ├── event/
│   │   │       │   ├── ContractEvent.java
│   │   │       │   ├── ContractEventListener.java
│   │   │       │   └── ContractEventPublisher.java
│   │   │       ├── util/
│   │   │       │   ├── PdfUtils.java
│   │   │       │   └── SignatureUtils.java
│   │   │       └── ContractServiceApplication.java
│   │   └── resources/
│   │       ├── application.yml
│   │       ├── bootstrap.yml
│   │       └── db/migration/
│   │           └── V1__init.sql
│   └── test/
│       └── java/
│           └── com/odc/contract/
├── pom.xml
└── README.md
```

## 3. API Endpoints

1. `GET /api/contracts` - Lấy danh sách hợp đồng
2. `GET /api/contracts/{id}` - Lấy thông tin chi tiết hợp đồng
3. `POST /api/contracts` - Tạo hợp đồng mới
4. `PUT /api/contracts/{id}` - Cập nhật thông tin hợp đồng
5. `POST /api/contracts/{id}/generate-pdf` - Tạo file PDF cho hợp đồng
6. `POST /api/contracts/{id}/sign` - Ký hợp đồng
7. `GET /api/contracts/{id}/download` - Tải xuống file PDF hợp đồng
8. `GET /api/contracts/{id}/verify-signature` - Xác minh chữ ký hợp đồng

## 4. Triển khai chi tiết

### 4.1. Model

```java
@Entity
@Table(name = "contracts")
@Data
@NoArgsConstructor
public class Contract {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private UUID id;
    
    @Column(nullable = false)
    private String title;
    
    @Column(columnDefinition = "TEXT")
    private String description;
    
    @Column(name = "business_id", nullable = false)
    private UUID businessId;
    
    @Column(name = "requirement_id")
    private UUID requirementId;
    
    @Column(name = "start_date", nullable = false)
    private LocalDate startDate;
    
    @Column(name = "end_date")
    private LocalDate endDate;
    
    @Column(name = "contract_type", nullable = false)
    private String contractType;
    
    @Column(nullable = false)
    private String status;
    
    @Column(name = "file_path")
    private String filePath;
    
    @Column(name = "signature_info")
    private String signatureInfo;
    
    @Column(name = "created_at", nullable = false, updatable = false)
    private LocalDateTime createdAt;
    
    @Column(name = "updated_at", nullable = false)
    private LocalDateTime updatedAt;
    
    @PrePersist
    protected void onCreate() {
        createdAt = LocalDateTime.now();
        updatedAt = LocalDateTime.now();
    }
    
    @PreUpdate
    protected void onUpdate() {
        updatedAt = LocalDateTime.now();
    }
}
```

### 4.2. Repository

```java
@Repository
public interface ContractRepository extends JpaRepository<Contract, UUID> {
    List<Contract> findByBusinessId(UUID businessId);
    List<Contract> findByStatus(String status);
    Optional<Contract> findByIdAndBusinessId(UUID id, UUID businessId);
}
```

### 4.3. Service

#### 4.3.1. ContractService

```java
public interface ContractService {
    List<ContractDTO> getAllContracts();
    ContractDTO getContractById(UUID id);
    List<ContractDTO> getContractsByBusinessId(UUID businessId);
    List<ContractDTO> getContractsByStatus(String status);
    ContractDTO createContract(ContractRequestDTO contractRequest);
    ContractDTO updateContract(UUID id, ContractRequestDTO contractRequest);
    void deleteContract(UUID id);
    byte[] generatePdf(UUID id);
    ContractDTO signContract(UUID id, byte[] signature);
    byte[] downloadContract(UUID id);
    boolean verifySignature(UUID id);
}
```

#### 4.3.2. PdfService

```java
public interface PdfService {
    byte[] generateContractPdf(Contract contract);
    byte[] addSignatureField(byte[] pdfBytes, String fieldName);
    byte[] signPdf(byte[] pdfBytes, byte[] signature, String fieldName);
    boolean verifySignature(byte[] pdfBytes);
}
```

#### 4.3.3. Triển khai PdfService với iText

```java
@Service
public class PdfServiceImpl implements PdfService {

    @Override
    public byte[] generateContractPdf(Contract contract) {
        try (ByteArrayOutputStream baos = new ByteArrayOutputStream()) {
            PdfWriter writer = new PdfWriter(baos);
            PdfDocument pdf = new PdfDocument(writer);
            Document document = new Document(pdf);
            
            // Thêm tiêu đề
            document.add(new Paragraph(contract.getTitle())
                    .setFontSize(20)
                    .setBold());
            
            // Thêm thông tin hợp đồng
            document.add(new Paragraph("Mô tả: " + contract.getDescription()));
            document.add(new Paragraph("Loại hợp đồng: " + contract.getContractType()));
            document.add(new Paragraph("Ngày bắt đầu: " + contract.getStartDate()));
            document.add(new Paragraph("Ngày kết thúc: " + contract.getEndDate()));
            
            // Thêm trường chữ ký
            PdfAcroForm form = PdfAcroForm.getAcroForm(pdf, true);
            Rectangle signatureRect = new Rectangle(100, 100, 200, 50);
            PdfFormField signature = PdfFormField.createSignature(pdf, signatureRect);
            signature.setFieldName("signature");
            form.addField(signature);
            
            document.close();
            return baos.toByteArray();
        } catch (Exception e) {
            throw new PdfGenerationException("Không thể tạo PDF: " + e.getMessage());
        }
    }

    @Override
    public byte[] addSignatureField(byte[] pdfBytes, String fieldName) {
        try (ByteArrayOutputStream baos = new ByteArrayOutputStream()) {
            PdfReader reader = new PdfReader(new ByteArrayInputStream(pdfBytes));
            PdfWriter writer = new PdfWriter(baos);
            PdfDocument pdf = new PdfDocument(reader, writer);
            
            PdfAcroForm form = PdfAcroForm.getAcroForm(pdf, true);
            Rectangle signatureRect = new Rectangle(100, 100, 200, 50);
            PdfFormField signature = PdfFormField.createSignature(pdf, signatureRect);
            signature.setFieldName(fieldName);
            form.addField(signature);
            
            pdf.close();
            return baos.toByteArray();
        } catch (Exception e) {
            throw new PdfGenerationException("Không thể thêm trường chữ ký: " + e.getMessage());
        }
    }

    @Override
    public byte[] signPdf(byte[] pdfBytes, byte[] signature, String fieldName) {
        try (ByteArrayOutputStream baos = new ByteArrayOutputStream()) {
            PdfReader reader = new PdfReader(new ByteArrayInputStream(pdfBytes));
            PdfSigner signer = new PdfSigner(reader, baos, new StampingProperties());
            
            IExternalSignature pks = new PrivateKeySignature(getPrivateKey(), DigestAlgorithms.SHA256, "BC");
            IExternalDigest digest = new BouncyCastleDigest();
            
            signer.signDetached(digest, pks, getCertificateChain(), null, null, null, 0, PdfSigner.CryptoStandard.CMS);
            
            return baos.toByteArray();
        } catch (Exception e) {
            throw new SignatureException("Không thể ký PDF: " + e.getMessage());
        }
    }

    @Override
    public boolean verifySignature(byte[] pdfBytes) {
        try {
            PdfReader reader = new PdfReader(new ByteArrayInputStream(pdfBytes));
            PdfDocument pdf = new PdfDocument(reader);
            SignatureUtil signUtil = new SignatureUtil(pdf);
            List<String> names = signUtil.getSignatureNames();
            
            if (names.isEmpty()) {
                return false;
            }
            
            for (String name : names) {
                PdfPKCS7 pkcs7 = signUtil.readSignatureData(name);
                if (!pkcs7.verifySignatureIntegrityAndAuthenticity()) {
                    return false;
                }
            }
            
            pdf.close();
            return true;
        } catch (Exception e) {
            throw new SignatureException("Không thể xác minh chữ ký: " + e.getMessage());
        }
    }
    
    private PrivateKey getPrivateKey() {
        // Lấy private key từ keystore
        // Triển khai thực tế sẽ phụ thuộc vào cách lưu trữ và quản lý key
        return null;
    }
    
    private Certificate[] getCertificateChain() {
        // Lấy chuỗi certificate từ keystore
        // Triển khai thực tế sẽ phụ thuộc vào cách lưu trữ và quản lý certificate
        return null;
    }
}
```

### 4.4. Controller

```java
@RestController
@RequestMapping("/api/contracts")
public class ContractController {

    private final ContractService contractService;
    
    @Autowired
    public ContractController(ContractService contractService) {
        this.contractService = contractService;
    }
    
    @GetMapping
    public ResponseEntity<List<ContractDTO>> getAllContracts() {
        return ResponseEntity.ok(contractService.getAllContracts());
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<ContractDTO> getContractById(@PathVariable UUID id) {
        return ResponseEntity.ok(contractService.getContractById(id));
    }
    
    @GetMapping("/business/{businessId}")
    public ResponseEntity<List<ContractDTO>> getContractsByBusinessId(@PathVariable UUID businessId) {
        return ResponseEntity.ok(contractService.getContractsByBusinessId(businessId));
    }
    
    @GetMapping("/status/{status}")
    public ResponseEntity<List<ContractDTO>> getContractsByStatus(@PathVariable String status) {
        return ResponseEntity.ok(contractService.getContractsByStatus(status));
    }
    
    @PostMapping
    public ResponseEntity<ContractDTO> createContract(@RequestBody ContractRequestDTO contractRequest) {
        return ResponseEntity.status(HttpStatus.CREATED).body(contractService.createContract(contractRequest));
    }
    
    @PutMapping("/{id}")
    public ResponseEntity<ContractDTO> updateContract(@PathVariable UUID id, @RequestBody ContractRequestDTO contractRequest) {
        return ResponseEntity.ok(contractService.updateContract(id, contractRequest));
    }
    
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteContract(@PathVariable UUID id) {
        contractService.deleteContract(id);
        return ResponseEntity.noContent().build();
    }
    
    @PostMapping("/{id}/generate-pdf")
    public ResponseEntity<byte[]> generatePdf(@PathVariable UUID id) {
        byte[] pdfBytes = contractService.generatePdf(id);
        return ResponseEntity.ok()
                .header(HttpHeaders.CONTENT_DISPOSITION, "attachment; filename=contract-" + id + ".pdf")
                .contentType(MediaType.APPLICATION_PDF)
                .body(pdfBytes);
    }
    
    @PostMapping("/{id}/sign")
    public ResponseEntity<ContractDTO> signContract(@PathVariable UUID id, @RequestBody byte[] signature) {
        return ResponseEntity.ok(contractService.signContract(id, signature));
    }
    
    @GetMapping("/{id}/download")
    public ResponseEntity<byte[]> downloadContract(@PathVariable UUID id) {
        byte[] pdfBytes = contractService.downloadContract(id);
        return ResponseEntity.ok()
                .header(HttpHeaders.CONTENT_DISPOSITION, "attachment; filename=contract-" + id + ".pdf")
                .contentType(MediaType.APPLICATION_PDF)
                .body(pdfBytes);
    }
    
    @GetMapping("/{id}/verify-signature")
    public ResponseEntity<Boolean> verifySignature(@PathVariable UUID id) {
        return ResponseEntity.ok(contractService.verifySignature(id));
    }
}
```

### 4.5. Tích hợp với Business Service

```java
@FeignClient(name = "business-service")
public interface BusinessServiceClient {
    @GetMapping("/api/businesses/{id}")
    ResponseEntity<BusinessDTO> getBusinessById(@PathVariable("id") UUID id);
    
    @GetMapping("/api/businesses/{id}/requirements/{requirementId}")
    ResponseEntity<RequirementDTO> getRequirementById(@PathVariable("id") UUID businessId, @PathVariable("requirementId") UUID requirementId);
}
```

### 4.6. Tích hợp với Notification Service (Kafka)

```java
@Component
public class ContractEventPublisher {

    private final KafkaTemplate<String, ContractEvent> kafkaTemplate;
    private final String topicName = "contract-events";
    
    @Autowired
    public ContractEventPublisher(KafkaTemplate<String, ContractEvent> kafkaTemplate) {
        this.kafkaTemplate = kafkaTemplate;
    }
    
    public void publishContractCreated(Contract contract) {
        ContractEvent event = new ContractEvent("CONTRACT_CREATED", contract.getId(), contract.getBusinessId());
        kafkaTemplate.send(topicName, event);
    }
    
    public void publishContractSigned(Contract contract) {
        ContractEvent event = new ContractEvent("CONTRACT_SIGNED", contract.getId(), contract.getBusinessId());
        kafkaTemplate.send(topicName, event);
    }
    
    public void publishContractUpdated(Contract contract) {
        ContractEvent event = new ContractEvent("CONTRACT_UPDATED", contract.getId(), contract.getBusinessId());
        kafkaTemplate.send(topicName, event);
    }
}
```

## 5. Lưu ý khi triển khai

### 5.1. Xử lý PDF và Chữ ký số

Có hai cách tiếp cận chính để xử lý chữ ký trong hợp đồng:

1. **Chữ ký điện tử đơn giản**: Chỉ thêm hình ảnh chữ ký vào PDF
   - Dễ triển khai
   - Không có giá trị pháp lý cao
   - Phù hợp cho các hợp đồng nội bộ

2. **Chữ ký số (Digital Signature)**: Sử dụng PKI (Public Key Infrastructure)
   - Có giá trị pháp lý cao
   - Đảm bảo tính toàn vẹn và xác thực của tài liệu
   - Yêu cầu quản lý khóa và chứng chỉ

Tùy vào yêu cầu cụ thể, có thể chọn một trong hai cách tiếp cận trên. Trong ví dụ triển khai, chúng ta đã sử dụng cách tiếp cận chữ ký số với iText.

### 5.2. Lưu trữ và Bảo mật

1. **Lưu trữ file PDF**:
   - Lưu trữ trên hệ thống file: Đơn giản nhưng khó mở rộng
   - Lưu trữ trên cloud storage (S3, GCS): Dễ mở rộng, có thể tích hợp với CDN
   - Lưu trữ trong database: Không khuyến nghị cho file lớn

2. **Bảo mật**:
   - Mã hóa file PDF khi lưu trữ
   - Sử dụng HTTPS cho tất cả các API
   - Xác thực và phân quyền cho các API
   - Logging đầy đủ các hoạt động liên quan đến hợp đồng

### 5.3. Quy trình xử lý hợp đồng

1. Tạo hợp đồng mới (draft)
2. Tạo file PDF từ thông tin hợp đồng
3. Thêm trường chữ ký vào PDF
4. Gửi hợp đồng cho các bên liên quan
5. Các bên ký hợp đồng
6. Xác minh chữ ký và cập nhật trạng thái hợp đồng
7. Lưu trữ hợp đồng đã ký

## 6. Kết luận

Contract Service là một microservice quan trọng trong hệ thống Lab-based ODC Platform, giúp quản lý hợp đồng giữa doanh nghiệp và nền tảng. Việc triển khai service này cần chú ý đến các vấn đề về bảo mật, tính toàn vẹn dữ liệu và giá trị pháp lý của hợp đồng.

Việc sử dụng các thư viện như iText hoặc PDFBox giúp đơn giản hóa quá trình tạo và xử lý PDF, trong khi việc tích hợp với các service khác như Business Service và Notification Service giúp đảm bảo tính nhất quán trong toàn hệ thống.