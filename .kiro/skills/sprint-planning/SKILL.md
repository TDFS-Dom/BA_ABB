---
name: sprint-planning
description: Lập kế hoạch sprint, theo dõi velocity, xác định rủi ro và phân bổ nguồn lực cho ngân hàng Việt Nam
---

# Skill Lập kế hoạch Sprint

## Khi nào dùng
Gọi cho sprint planning sessions, phân tích velocity, hoặc quyết định phân bổ nguồn lực.

## Quy trình Sprint Planning
1. Review sprint trước: velocity, carry-over, blockers
2. Đánh giá năng lực team: nghỉ phép, trực ca, đào tạo
3. Kéo stories từ backlog đã ưu tiên
4. Xác định rủi ro và dependencies
5. Cam kết sprint goal

## Theo dõi Velocity
- Theo dõi story points hoàn thành mỗi sprint (trung bình 3 sprint gần nhất)
- Cảnh báo nếu velocity giảm > 20% so với trung bình
- Điều chỉnh cam kết khi team thay đổi (thành viên mới ramp 50% sprint đầu)

## Template xác định rủi ro
| Rủi ro | Khả năng | Tác động | Giảm thiểu | Người chịu TN |
|--------|----------|----------|------------|----------------|
| [Mô tả] | Cao/TB/Thấp | Cao/TB/Thấp | [Hành động] | [Tên] |

## Lưu ý Sprint đặc thù ngân hàng Việt Nam
- Deadline NHNN không thương lượng — luôn dự trữ năng lực
- Trực production: 1 developer on-call mỗi sprint
- Tết Nguyên đán: code freeze 2 tuần trước, giảm 50% cam kết sprint trước Tết
- Cuối quý tài chính: không deploy tuần cuối quý
- Mùa kiểm toán (Q1, Q3): dự trữ 10-15% năng lực cho hỗ trợ kiểm toán
- Ngày lễ Việt Nam (30/4, 1/5, 2/9, Giỗ Tổ): tính vào capacity planning
