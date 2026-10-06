# Metrics Pack — Agent luyện tập tư vấn cho sales xe VinFast mới

**Học viên:** Phạm Quốc Đạt · **MHV:** 2A202602384  
**Use case chính:** Sales mới giải ca tư vấn mô phỏng trong lộ trình onboarding.  
**Phạm vi:** Thiết kế đo lường, chưa phải kết quả đo. Lịch ca học, rubric và nguồn chính sách cần được đội đào tạo xác nhận trước khi triển khai.

## 00 — Dự án, persona, core job

- **Dự án:** Agent giúp sales xe tra cứu thông tin, hỗ trợ tư vấn và luyện tập kiến thức. Chính sách có thể đổi thường xuyên, trong khi tri thức sẵn có của LLM có thể lỗi thời; bài tập và phản hồi cần dẫn nguồn đã duyệt, phiên bản và ngày hiệu lực.
- **Persona:** Một nhân viên sales xe VinFast mới. Họ cần học nhiều kiến thức và thực hành tư vấn trong khi quản lý phụ trách nhiều nhân viên, khó kèm cặp riêng từng ca.
- **Core job, diễn đạt bằng lời người dùng:** “Khi học tư vấn xe và chính sách hiện hành, tôi cần thử xử lý tình huống khách hàng, biết mình đúng/sai ở đâu và sửa được trước khi gặp khách thật.”
- **Use case duy nhất:** Luyện tập bằng ca tư vấn mô phỏng. Tra cứu và hỏi Agent chỉ hỗ trợ quá trình này; không được tính thay cho hành vi cốt lõi.

**Giả định cần xác nhận:** Mỗi ca học có module, tình huống được giao và hạn hoàn thành; đề/rubric có phiên bản; nguồn chính sách có trạng thái duyệt và ngày hiệu lực. Nếu dữ liệu này chưa có, phải bổ sung trước khi tính metric.

## 01 — Core Action

### Bốn khái niệm

| Khái niệm | Định nghĩa trong use case |
|---|---|
| Core job | Tập xử lý tình huống tư vấn để biết và sửa lỗ hổng kiến thức. |
| Core action | **Sales mới hoàn thành lời tư vấn cho một ca mô phỏng được giao và nộp bài.** Học viên đã chọn “giải ca tư vấn mô phỏng”. |
| Core value | Nhận phản hồi có căn cứ về phần áp dụng đúng/sai và cách cải thiện. |
| Core value event | Sales xem phản hồi của bài đã được chấm hợp lệ; sự tiến bộ tiếp tục được kiểm chứng ở lần sửa hoặc ca sau. |

### Core Action Card

| Thành phần | Định nghĩa |
|---|---|
| Target user / actor | Sales xe VinFast mới trong onboarding. |
| Core job | Tập tư vấn tình huống, nhận ra và sửa lỗi trước khi gặp khách thật. |
| Core action | Hoàn thành và nộp lời tư vấn cho một ca mô phỏng được giao. |
| Object | `simulation_case_id` và `case_version`, thuộc một `learning_module_id`. |
| Preconditions | Sales thuộc lộ trình; ca được giao; đề, rubric và nguồn chính sách được duyệt, còn hiệu lực. |
| Completion rule | Server xác thực đủ các phần bắt buộc, lưu bài ở trạng thái `submitted` với `attempt_id` duy nhất. Gõ dở, mở ca hoặc nhấn gửi khi bài chưa hợp lệ không tính. |
| Core value | Biết điểm đúng/sai và cách sửa theo nguồn hiện hành. |
| Evidence of value | Assessment hợp lệ đã sinh phản hồi và sales mở xem sau khi chấm. |
| Candidate events | `simulation_response_submitted` (core action), `simulation_feedback_viewed` (first value). |

**Rationale bản nháp do AI soạn:** Bài nộp đòi hỏi sales áp dụng kiến thức vào bối cảnh khách hàng; có đối tượng và điểm hoàn tất quan sát được; có thể chấm và phản hồi. `app_opened`, `login_completed`, `chat_message_sent` hay câu trả lời do LLM tự tạo không chứng minh sales đã luyện kỹ năng.

### Tự kiểm 5 tiêu chí

| Tiêu chí | Kết quả | Bằng chứng |
|---|---|---|
| Gần core value | Đạt | Bài nộp là đầu vào trực tiếp cho phản hồi. |
| Lặp lại được | Đạt | Mỗi module/ca mới là một cơ hội luyện tập có ý nghĩa. |
| Quan sát được | Đạt | Server ghi `attempt_id`, trạng thái và thời điểm nộp. |
| Có ý nghĩa | Đạt | Sales phải hình thành lời tư vấn cho một tình huống. |
| Có thể tác động | Đạt | Team có thể cải thiện đề, rubric, phản hồi, lộ trình và nguồn. |

**GATE 1:** Actor, object, completion rule rõ; đạt 5/5 tiêu chí theo thiết kế. Cần kiểm tra lại với người dùng thử khi có prototype.

## 02 — Action Nature & cadence

| Thành phần | Phân tích |
|---|---|
| Actor | Sales mới được giao module onboarding. |
| Intent | Luyện áp dụng kiến thức và biết mình có thể tư vấn đúng chưa. |
| Trigger | Ca học/module được giao; đôi khi chính sách mới tạo ca ôn bổ sung. Notification chỉ nhắc, không tạo nhu cầu học từ số 0. |
| Effort | Đọc bối cảnh, tra nguồn, suy luận và soạn lời tư vấn; cao hơn một lượt hỏi đáp. |
| Value timing | Phản hồi đến sau khi nộp/chấm; năng lực tích lũy qua nhiều ca. |
| State | Tiến độ module, bài nộp, điểm mạnh/yếu và nội dung cần ôn được lưu. |
| Dependency | Đề, rubric, nguồn được duyệt và lịch onboarding. |
| Repeat condition | Có ca/module tiếp theo hoặc ca luyện lại để sửa điểm yếu; lần lặp phải có tình huống hay mục tiêu mới. |

**Dạng hành vi:** Tiến trình học tích lũy theo ca học/onboarding, đôi khi phản ứng với chính sách mới; không mặc định là daily habit.

**Kết luận cadence bản nháp do AI diễn đạt từ lựa chọn của học viên:** Đối với sales mới, giải ca tư vấn mô phỏng thường xuất hiện khi có ca học/module onboarding hoặc nội dung mới cần ôn, **vì** một ca cần thời gian chuẩn bị, áp dụng kiến thức và nhận phản hồi. Do đó, nhịp đo phù hợp là **theo ca học được giao**, ở cấp **mỗi sales trong từng module**; xem xu hướng qua các module theo cohort onboarding. Học viên cần viết lại phần lý do theo thực tế đội đào tạo để đúng quy tắc lab.

**GATE 2:** Có câu “vì”, cadence dựa trên điều kiện lặp của hành vi; không dùng DAU/WAU chỉ vì dashboard có sẵn.

## 03 — Metric System

### Quy ước để tính được

- **Eligible learner:** sales mới thuộc cohort onboarding, loại bot, tài khoản test/nội bộ.
- **Eligible module:** module đã được giao, có `module_start_at`, `module_due_at`, đề/rubric hợp lệ; module hủy hoặc chưa đến hạn không vào mẫu số completion.
- **Valid submission:** `simulation_response_submitted` đủ nội dung, đúng phiên bản, được assessment xác nhận `is_valid=true`.
- **Quality pass:** valid submission đạt ngưỡng rubric **được đội đào tạo duyệt**, không có lỗi chính sách trọng yếu và `source_status=current`. Chưa có rubric thật nên không tự đặt ngưỡng 80%.
- **Đếm duy nhất:** NSM tối đa một lần cho cặp `learner_id × case_id × case_version`; retry là attempt khác nhưng không tăng số ca đạt chuẩn.

### Activation

| Thành phần | Định nghĩa |
|---|---|
| Start event | `learning_path_started`: sales thực sự bắt đầu lộ trình được giao. |
| Activation event | `simulation_feedback_viewed` lần đầu cho một valid submission đã được chấm. Sales đã thấy mình đúng/sai ở đâu. |
| Time window | Từ start event đến `module_due_at` của **ca học đầu tiên**. Chỉ đánh giá cohort khi hạn ca đầu đã qua; lịch thay đổi phải có phiên bản. |

**Activation rate:** số eligible learners xem phản hồi hợp lệ trong window / số eligible learners bắt đầu lộ trình và có ca đầu tiên đã đến hạn. Phân tổ theo cohort bắt đầu và module đầu. Đăng nhập hoặc xem onboarding chưa phải activation.

### Engagement — tối đa hai góc đo

1. **Frequency theo module — module participation rate:** số eligible modules có ít nhất một valid submission trước hạn / số eligible modules đã đến hạn của sales. Retry trong cùng module không tăng tử số.
2. **Depth — remediation completion rate:** trong các ca lần đầu chưa đạt, tỷ lệ ca mà sales xem phản hồi rồi nộp bản sửa hợp lệ trước hạn sửa. Người đạt ngay lần đầu không bị ép retry.

### North Star Metric

**NSM: Số ca mô phỏng đạt chuẩn tư vấn trên mỗi ca học được giao**, báo cáo theo cohort và module.

- **Unit of value:** một ca mô phỏng do sales hoàn thành và được chấm.
- **Quality threshold:** quality pass theo rubric đã duyệt, không lỗi chính sách trọng yếu, nguồn còn hiệu lực.
- **Frequency:** mỗi ca học/module được giao, không phải mỗi ngày mở app.
- **Công thức:** đếm cặp sales–ca có ít nhất một assessment đạt chuẩn trước `module_due_at`; trình bày kèm số eligible modules và số sales, tránh ảo giác tăng trưởng khi chỉ tăng số đề giao.

### Leading indicators — tối đa ba

| Chỉ số | Cách tính | Giả thuyết dự báo |
|---|---|---|
| First-session activation rate | Theo định nghĩa activation. | Nhận phản hồi sớm cho sales điểm cần sửa ở ca sau. |
| Feedback review rate | Assessments hợp lệ có phản hồi được xem trước hạn / assessments hợp lệ có phản hồi sẵn. | Xem phản hồi nối kết kết quả ca trước với ca sau. |
| Remediation completion rate | Theo engagement depth. | Sales dùng phản hồi để thử lại thay vì chỉ nhận điểm. |

Đây là giả thuyết cần kiểm chứng bằng dữ liệu; chưa khẳng định quan hệ nhân quả.

### Counter-metric

**Stale-source assessment rate:** số assessments có `source_status=stale` hoặc `unknown` / tổng số assessments. Tách hai trạng thái khi báo cáo. Chỉ số này cảnh báo việc tăng số ca đạt hoặc số người quay lại trong khi nguồn chính sách/rubric lỗi thời; mục tiêu vận hành phải do đội đào tạo quyết định, không tự bịa.

**GATE 3, metric:** Activation đủ start/activation/window; NSM đủ unit/quality/frequency; có leading indicators và counter-metric.

## 04 — Retention Definition

**Metric:** *Next-module qualified return rate*. Quay lại nghĩa là hoàn thành một ca mô phỏng hợp lệ ở module onboarding kế tiếp, không phải mở app hoặc nhắn Agent.

| Thành phần | Định nghĩa |
|---|---|
| Unit | Một `learner_id` là sales mới. |
| Cohort entry | `simulation_feedback_viewed` hợp lệ đầu tiên sau valid submission ở module đầu; lưu `cohort_module_id` và thời điểm activation. |
| Return event | Valid `simulation_response_submitted` cho ca khác thuộc **module được giao tiếp theo**, ghép assessment `is_valid=true`. Không đòi pass để đo quay lại; pass ở NSM. |
| Window | Từ `next_module_start_at` đến `next_module_due_at` theo lịch của cohort; không gắn D7 cho mọi người. |
| Threshold | Ít nhất **một ca mô phỏng hợp lệ** ở module kế tiếp. Retry cùng ca không tính thêm. |
| Segment | Cohort onboarding, module đầu/tiếp theo, đội/địa điểm đào tạo nếu được phép; có thể tách module kiến thức sản phẩm và chính sách. |

**Công thức:** activated learners có ít nhất một valid submission ở module kế tiếp trong window / activated learners đã được giao module kế tiếp và module đó đã đến hạn. Xem thêm tỷ lệ đạt chuẩn ở module kế tiếp để không nhầm “quay lại” với “đã hiểu đúng”.

**Mốc so:** nhịp module tự nhiên; cohort/segment tương đương; benchmark sản phẩm cùng loại chỉ khi có nguồn. Chưa có baseline nên không đưa mục tiêu phần trăm hay D7 retention giả định.

**GATE 3, retention:** Đủ 6 thành phần và window khớp cadence ở mục 02.

## 05 — Product Loop

**Loại loop:** *Progress loop* của lộ trình học. Lịch module là natural trigger; notification chỉ có thể nhắc lịch.

```text
Chu kỳ 1:
Module được giao → sales giải và nộp ca → xem phản hồi theo nguồn hiện hành
→ lưu điểm mạnh/yếu và tiến độ → nhận ca tiếp theo phù hợp điểm cần luyện

Chu kỳ 2:
Module kế tiếp hoặc tình huống chính sách mới → sales giải ca khác
→ áp dụng điều vừa sửa → xem phản hồi và tiến bộ → lưu điểm cần luyện tiếp
```

**Metric hypothesis, bản nháp do AI viết theo yêu cầu của học viên:** “Nếu vòng lặp phản hồi → ca luyện tập tiếp theo hoạt động, *next-module qualified return rate* sẽ tăng qua **hai chu kỳ module onboarding tiếp theo**, vì sales thấy lỗi cụ thể và có ca mới để áp dụng điều vừa sửa.” Metric được định nghĩa ở mục 04. Chưa nêu mức tăng vì không có baseline. Học viên cần tự viết lại giả thuyết này trước khi nộp theo quy tắc lab.

**Cách kiểm:** Theo dõi các cohort đủ điều kiện qua các module, xem NSM và counter-metric cùng retention. Nếu so cohort có/không có loop, cần kiểm tra sự khác biệt lịch, độ khó đề và nguồn chính sách trước khi suy ra tác động.

**GATE 4, loop:** Hai chu kỳ, có saved state và lý do quay lại ngoài notification; hypothesis trỏ về metric ở mục 04.

## 06 — Tracking nhanh

### Năm core events

| Event | Điều đã xảy ra và thời điểm phát | Metric sử dụng |
|---|---|---|
| `learning_path_started` | Sales bắt đầu lộ trình được giao lần đầu; không phát khi chỉ tạo tài khoản. Ghi learner/path/cohort/time. | Activation start, cohort. |
| `learning_module_assigned` | Module có lịch, đề và rubric hợp lệ đã được giao; ghi module, start/due, schedule version. | Mẫu số activation/engagement, window retention, NSM theo module. |
| `simulation_response_submitted` | Server xác thực đủ nội dung và lưu một `attempt_id` ở trạng thái submitted; không phát khi lưu thất bại. | Core action, engagement; ghép assessment cho retention/NSM. |
| `simulation_assessment_completed` | Bài đã được chấm; ghi rubric version, `is_valid`, `passed`, lỗi chính sách trọng yếu, `source_status`, thời điểm. | Valid submission, NSM, retention, counter-metric. |
| `simulation_feedback_viewed` | Sales mở xem phản hồi của assessment hợp lệ; một event logic cho lượt xem đầu của assessment. | Activation, feedback review, cohort entry, remediation. |

**Đối chiếu hai chiều:** Mỗi event trên là đầu vào trực tiếp cho ít nhất một metric ở mục 03–04. NSM/retention cần ghép learner–module–case–attempt–assessment; activation cần start và feedback; counter-metric cần assessment. Không track mọi click.

### Acceptance criteria

1. **Chỉ phát sau khi hoàn tất:** Với một `attempt_id`, khi server chuyển `draft → submitted` sau khi kiểm tra nội dung, có đúng một `simulation_response_submitted`. Bấm gửi khi thiếu trường, lưu lỗi hoặc reload trang không tạo event.
2. **Retry không nhân đôi:** Client gửi lại cùng idempotency key sau mất mạng chỉ tạo một event logic. Lần sửa thật có `attempt_id` mới, nhưng NSM/retention vẫn không đếm trùng cặp learner–case trong một module.
3. **Kiểm soát phiên bản nguồn:** Assessment lưu `rubric_version`, `source_bundle_version`, ngày hiệu lực và `assessed_at`. Nguồn stale/unknown không đạt quality pass nhưng vẫn vào mẫu số counter-metric; cập nhật nguồn sau đó không âm thầm viết lại kết quả lịch sử.

**Contract tối thiểu:** learner ID ổn định; path/module/case/attempt IDs; event schema có phiên bản; timestamp UTC và timezone báo cáo thống nhất; lọc bot/test/internal. Bản Metric Definition Contract đầy đủ có thể bổ sung ngoài giờ lab.

**GATE 4, tracking:** 5 events đều map được, có 3 acceptance criteria kiểm thử được.

## Phase 5 — Tự soi lỗi, revision và reflection

| Câu hỏi | Kết quả bản nháp |
|---|---|
| Core action là thao tác giao diện hoặc output hệ thống? | Không: sales hoàn thành và nộp lời tư vấn; câu LLM sinh không tính thay. |
| Activation là đăng nhập/xem hết onboarding? | Không: phải nộp bài hợp lệ và xem phản hồi. |
| Frequency cao hơn nhu cầu thật? | Không ép daily/weekly; đo theo module được giao, cần đối chiếu lịch thật. |
| Loop có reason to return ngoài notification? | Có: điểm yếu cần sửa, tiến bộ được lưu và ca kế tiếp có mục tiêu luyện tập. |
| Retention dùng một window cho mọi cadence? | Không: window theo lịch module tiếp theo của cohort. |
| Mọi event map về metric? | Có: bảng event ghi metric hoặc phép đối soát. |
| Mỗi metric có event để tính? | Có theo thiết kế và phép ghép định danh nêu trên. |

**Revision/rationale, bản nháp AI:** Các hành vi “hỏi Agent”, “mở tài liệu” dễ track nhưng không chứng minh sales áp dụng kiến thức. Metrics Pack chốt “giải ca tư vấn mô phỏng” theo lựa chọn của học viên. Không đặt ngưỡng đỗ, benchmark retention hoặc lịch daily/weekly khi thiếu nguồn thực tế.

**Điều áp dụng vào dự án thật:** Xây ca mô phỏng có rubric và nguồn chính sách theo phiên bản; chỉ phát event core action khi server lưu bài hoàn tất; phản hồi có căn cứ và lưu điểm cần luyện cho module kế tiếp. Sau pilot, kiểm tra xem việc xem phản hồi và sửa bài có dự báo chất lượng ở module tiếp theo không.

**Lưu ý trước khi nộp:** Học viên phải tự rà và viết lại lý do chọn core action, câu cadence, metric hypothesis, revision/rationale và reflection bằng phán đoán của mình theo quy tắc AI trong lab. Không trình bày các giả định lịch, rubric hay chính sách ở đây như dữ liệu đã kiểm chứng.
