# Luồng xử lý hợp đồng và chữ ký số

## 1. Tổng quan về luồng xử lý

Luồng xử lý hợp đồng và chữ ký số trong hệ thống Lab-based ODC Platform bao gồm các bước sau:

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│  Business       │     │  Contract       │     │  Notification   │     │   User           │
│  Service        │────▶│  Service        │────▶│  Service        │────▶│  (Business)     │
└─────────────────┘     └─────────────────┘     └─────────────────┘     └─────────────────┘
        │                        ▲                                               │
        │                        │                                               │
        └────────────────────────┴───────────────────────────────────────────────┘
```

## 2. Chi tiết các bước trong luồng xử lý

### 2.1. Tạo hợp đồng

1. **Business Service** tạo yêu cầu hợp đồng mới dựa trên thông tin dự án/yêu cầu
2. **Business Service** gọi API của **Contract Service** để tạo hợp đồng mới
3. **Contract Service** lưu thông tin hợp đồng vào database
4. **Contract Service** tạo file PDF từ thông tin hợp đồng
5. **Contract Service** thêm trường chữ ký vào PDF
6. **Contract Service** lưu trữ file PDF (local/cloud storage)
7. **Contract Service** cập nhật đường dẫn file trong database
8. **Contract Service** gửi sự kiện "CONTRACT_CREATED" qua Kafka
9. **Notification Service** nhận sự kiện và gửi thông báo cho doanh nghiệp

### 2.2. Ký hợp đồng

1. **User (Business)** nhận thông báo về hợp đồng cần ký
2. **User (Business)** truy cập hệ thống và xem hợp đồng
3. **User (Business)** tải xuống hợp đồng (nếu cần)
4. **User (Business)** ký hợp đồng bằng một trong các phương thức:
   - Ký trực tiếp trên giao diện web (sử dụng canvas để vẽ chữ ký)
   - Tải lên hình ảnh chữ ký
   - Sử dụng chữ ký số (digital certificate)
5. **Business Service** gửi yêu cầu ký hợp đồng đến **Contract Service**
6. **Contract Service** xử lý chữ ký và thêm vào PDF
7. **Contract Service** xác minh tính hợp lệ của chữ ký (nếu sử dụng chữ ký số)
8. **Contract Service** cập nhật trạng thái hợp đồng thành "SIGNED"
9. **Contract Service** gửi sự kiện "CONTRACT_SIGNED" qua Kafka
10. **Notification Service** nhận sự kiện và gửi thông báo xác nhận cho các bên liên quan

### 2.3. Xác minh và truy xuất hợp đồng

1. **User (Business)** hoặc admin có thể truy xuất hợp đồng đã ký
2. **Business Service** gọi API của **Contract Service** để lấy thông tin hợp đồng
3. **Contract Service** trả về thông tin hợp đồng và đường dẫn đến file PDF
4. **User (Business)** có thể tải xuống file PDF hoặc xem trực tiếp trên hệ thống
5. **Contract Service** cung cấp API để xác minh tính hợp lệ của chữ ký (nếu cần)

## 3. Triển khai chi tiết

### 3.1. Tạo hợp đồng

```java
// BusinessService gọi ContractService
@PostMapping("/api/businesses/{businessId}/requirements/{requirementId}/contracts")
public ResponseEntity<ContractDTO> createContract(
        @PathVariable UUID businessId,
        @PathVariable UUID requirementId,
        @RequestBody ContractRequestDTO contractRequest) {
    
    // Kiểm tra business và requirement tồn tại
    BusinessDTO business = businessService.getBusinessById(businessId);
    RequirementDTO requirement = requirementService.getRequirementById(requirementId);
    
    // Tạo contract request
    ContractRequestDTO request = new ContractRequestDTO();
    request.setBusinessId(businessId);
    request.setRequirementId(requirementId);
    request.setTitle(contractRequest.getTitle());
    request.setDescription(contractRequest.getDescription());
    request.setStartDate(contractRequest.getStartDate());
    request.setEndDate(contractRequest.getEndDate());
    request.setContractType(contractRequest.getContractType());
    
    // Gọi Contract Service thông qua Feign Client
    ResponseEntity<ContractDTO> response = contractServiceClient.createContract(request);
    
    return response;
}

// ContractService xử lý yêu cầu
@Service
public class ContractServiceImpl implements ContractService {

    private final ContractRepository contractRepository;
    private final PdfService pdfService;
    private final ContractEventPublisher eventPublisher;
    
    @Autowired
    public ContractServiceImpl(ContractRepository contractRepository, 
                              PdfService pdfService,
                              ContractEventPublisher eventPublisher) {
        this.contractRepository = contractRepository;
        this.pdfService = pdfService;
        this.eventPublisher = eventPublisher;
    }
    
    @Override
    @Transactional
    public ContractDTO createContract(ContractRequestDTO contractRequest) {
        // Tạo đối tượng Contract từ request
        Contract contract = new Contract();
        contract.setBusinessId(contractRequest.getBusinessId());
        contract.setRequirementId(contractRequest.getRequirementId());
        contract.setTitle(contractRequest.getTitle());
        contract.setDescription(contractRequest.getDescription());
        contract.setStartDate(contractRequest.getStartDate());
        contract.setEndDate(contractRequest.getEndDate());
        contract.setContractType(contractRequest.getContractType());
        contract.setStatus("DRAFT");
        
        // Lưu contract vào database
        Contract savedContract = contractRepository.save(contract);
        
        // Tạo file PDF
        byte[] pdfBytes = pdfService.generateContractPdf(savedContract);
        
        // Thêm trường chữ ký
        pdfBytes = pdfService.addSignatureField(pdfBytes, "signature");
        
        // Lưu file PDF
        String filePath = saveContractPdf(savedContract.getId(), pdfBytes);
        
        // Cập nhật đường dẫn file
        savedContract.setFilePath(filePath);
        savedContract = contractRepository.save(savedContract);
        
        // Gửi sự kiện
        eventPublisher.publishContractCreated(savedContract);
        
        // Trả về DTO
        return mapToDTO(savedContract);
    }
    
    private String saveContractPdf(UUID contractId, byte[] pdfBytes) {
        // Triển khai lưu file PDF (local/cloud storage)
        // Ví dụ: lưu vào thư mục local
        String fileName = "contract-" + contractId + ".pdf";
        String filePath = "contracts/" + fileName;
        
        try {
            Files.createDirectories(Paths.get("contracts"));
            Files.write(Paths.get(filePath), pdfBytes);
            return filePath;
        } catch (IOException e) {
            throw new PdfGenerationException("Không thể lưu file PDF: " + e.getMessage());
        }
    }
    
    private ContractDTO mapToDTO(Contract contract) {
        ContractDTO dto = new ContractDTO();
        dto.setId(contract.getId());
        dto.setBusinessId(contract.getBusinessId());
        dto.setRequirementId(contract.getRequirementId());
        dto.setTitle(contract.getTitle());
        dto.setDescription(contract.getDescription());
        dto.setStartDate(contract.getStartDate());
        dto.setEndDate(contract.getEndDate());
        dto.setContractType(contract.getContractType());
        dto.setStatus(contract.getStatus());
        dto.setFilePath(contract.getFilePath());
        dto.setCreatedAt(contract.getCreatedAt());
        dto.setUpdatedAt(contract.getUpdatedAt());
        return dto;
    }
}
```

### 3.2. Ký hợp đồng

```java
// BusinessService gọi ContractService
@PostMapping("/api/businesses/{businessId}/contracts/{contractId}/sign")
public ResponseEntity<ContractDTO> signContract(
        @PathVariable UUID businessId,
        @PathVariable UUID contractId,
        @RequestBody SignatureRequestDTO signatureRequest) {
    
    // Kiểm tra business và contract tồn tại
    BusinessDTO business = businessService.getBusinessById(businessId);
    
    // Gọi Contract Service thông qua Feign Client
    ResponseEntity<ContractDTO> response = contractServiceClient.signContract(contractId, signatureRequest.getSignature());
    
    return response;
}

// ContractService xử lý yêu cầu
@Service
public class ContractServiceImpl implements ContractService {

    // ... các phương thức khác ...
    
    @Override
    @Transactional
    public ContractDTO signContract(UUID id, byte[] signature) {
        // Tìm contract
        Contract contract = contractRepository.findById(id)
                .orElseThrow(() -> new ContractNotFoundException("Không tìm thấy hợp đồng với ID: " + id));
        
        // Kiểm tra trạng thái
        if ("SIGNED".equals(contract.getStatus())) {
            throw new IllegalStateException("Hợp đồng đã được ký");
        }
        
        // Đọc file PDF
        byte[] pdfBytes = readContractPdf(contract.getFilePath());
        
        // Ký PDF
        byte[] signedPdfBytes = pdfService.signPdf(pdfBytes, signature, "signature");
        
        // Lưu file PDF đã ký
        String filePath = saveContractPdf(contract.getId(), signedPdfBytes);
        
        // Cập nhật thông tin contract
        contract.setFilePath(filePath);
        contract.setStatus("SIGNED");
        contract.setSignatureInfo("Signed at " + LocalDateTime.now());
        Contract savedContract = contractRepository.save(contract);
        
        // Gửi sự kiện
        eventPublisher.publishContractSigned(savedContract);
        
        // Trả về DTO
        return mapToDTO(savedContract);
    }
    
    private byte[] readContractPdf(String filePath) {
        try {
            return Files.readAllBytes(Paths.get(filePath));
        } catch (IOException e) {
            throw new RuntimeException("Không thể đọc file PDF: " + e.getMessage());
        }
    }
}
```

### 3.3. Xử lý sự kiện và thông báo

```java
// ContractEventPublisher
@Component
public class ContractEventPublisher {

    private final KafkaTemplate<String, ContractEvent> kafkaTemplate;
    private final String topicName = "contract-events";
    
    @Autowired
    public ContractEventPublisher(KafkaTemplate<String, ContractEvent> kafkaTemplate) {
        this.kafkaTemplate = kafkaTemplate;
    }
    
    public void publishContractCreated(Contract contract) {
        ContractEvent event = new ContractEvent();
        event.setEventType("CONTRACT_CREATED");
        event.setContractId(contract.getId());
        event.setBusinessId(contract.getBusinessId());
        event.setTimestamp(LocalDateTime.now());
        
        kafkaTemplate.send(topicName, event);
    }
    
    public void publishContractSigned(Contract contract) {
        ContractEvent event = new ContractEvent();
        event.setEventType("CONTRACT_SIGNED");
        event.setContractId(contract.getId());
        event.setBusinessId(contract.getBusinessId());
        event.setTimestamp(LocalDateTime.now());
        
        kafkaTemplate.send(topicName, event);
    }
}

// NotificationService xử lý sự kiện
@Component
public class ContractEventListener {

    private final NotificationService notificationService;
    
    @Autowired
    public ContractEventListener(NotificationService notificationService) {
        this.notificationService = notificationService;
    }
    
    @KafkaListener(topics = "contract-events", groupId = "notification-service")
    public void handleContractEvent(ContractEvent event) {
        switch (event.getEventType()) {
            case "CONTRACT_CREATED":
                sendContractCreatedNotification(event);
                break;
            case "CONTRACT_SIGNED":
                sendContractSignedNotification(event);
                break;
            default:
                // Xử lý các loại sự kiện khác
                break;
        }
    }
    
    private void sendContractCreatedNotification(ContractEvent event) {
        NotificationDTO notification = new NotificationDTO();
        notification.setRecipientId(event.getBusinessId());
        notification.setType("CONTRACT");
        notification.setTitle("Hợp đồng mới");
        notification.setContent("Một hợp đồng mới đã được tạo và đang chờ ký.");
        notification.setData(Map.of("contractId", event.getContractId().toString()));
        
        notificationService.sendNotification(notification);
    }
    
    private void sendContractSignedNotification(ContractEvent event) {
        NotificationDTO notification = new NotificationDTO();
        notification.setRecipientId(event.getBusinessId());
        notification.setType("CONTRACT");
        notification.setTitle("Hợp đồng đã được ký");
        notification.setContent("Hợp đồng đã được ký thành công.");
        notification.setData(Map.of("contractId", event.getContractId().toString()));
        
        notificationService.sendNotification(notification);
    }
}
```

## 4. Lưu ý về bảo mật và hiệu suất

### 4.1. Bảo mật

1. **Mã hóa dữ liệu**:
   - Mã hóa file PDF khi lưu trữ
   - Sử dụng HTTPS cho tất cả các API
   - Bảo vệ khóa và chứng chỉ số

2. **Xác thực và phân quyền**:
   - Chỉ cho phép người dùng có quyền thích hợp truy cập hợp đồng
   - Xác minh danh tính người ký
   - Ghi log đầy đủ các hoạt động liên quan đến hợp đồng

3. **Tính toàn vẹn dữ liệu**:
   - Sử dụng chữ ký số để đảm bảo tính toàn vẹn của hợp đồng
   - Xác minh chữ ký khi truy xuất hợp đồng

### 4.2. Hiệu suất

1. **Xử lý PDF**:
   - Tối ưu kích thước file PDF
   - Xử lý bất đồng bộ cho các tác vụ tạo và ký PDF

2. **Lưu trữ**:
   - Sử dụng cloud storage cho khả năng mở rộng
   - Cân nhắc CDN cho việc truy xuất file nhanh chóng

3. **Xử lý sự kiện**:
   - Sử dụng Kafka để xử lý sự kiện bất đồng bộ
   - Đảm bảo khả năng mở rộng của hệ thống thông báo

## 5. Kết luận

Luồng xử lý hợp đồng và chữ ký số là một phần quan trọng trong hệ thống Lab-based ODC Platform, giúp tự động hóa quá trình tạo, ký và quản lý hợp đồng giữa doanh nghiệp và nền tảng. Việc triển khai đúng luồng xử lý này sẽ giúp đảm bảo tính pháp lý, bảo mật và hiệu quả của hệ thống.

Contract Service là một supporting service trong kiến trúc microservices, tương tự như Notification Service, có nhiệm vụ hỗ trợ các core service khác như Business Service, Project Service trong việc xử lý các chức năng liên quan đến hợp đồng và chữ ký số.