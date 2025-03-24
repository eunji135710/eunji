import heapq
import random
import copy

# 퍼즐 목표 상태
goal_state = [
    [1, 2, 3, 4, 5],
    [6, 7, 8, 9, 0]
]

# 맨해튼 거리 휴리스틱 함수
def heuristic(state):
    distance = 0
    for i in range(2):
        for j in range(5):
            value = state[i][j]
            if value == 0:
                continue
            goal_i = (value - 1) // 5
            goal_j = (value - 1) % 5
            distance += abs(i - goal_i) + abs(j - goal_j)
    return distance

# 0(빈칸) 위치 찾기
def find_zero(state):
    for i in range(2):
        for j in range(5):
            if state[i][j] == 0:
                return i, j

# 인접 상태(이동 가능한 경우) 만들기
def get_neighbors(state):
    zero_i, zero_j = find_zero(state)
    moves = [(-1, 0), (1, 0), (0, -1), (0, 1)]  # 상, 하, 좌, 우
    neighbors = []

    for di, dj in moves:
        ni, nj = zero_i + di, zero_j + dj
        if 0 <= ni < 2 and 0 <= nj < 5:
            new_state = copy.deepcopy(state)
            new_state[zero_i][zero_j], new_state[ni][nj] = new_state[ni][nj], new_state[zero_i][zero_j]
            neighbors.append(new_state)
    return neighbors

# 상태를 문자열로 바꿔서 visited 체크용
def state_to_str(state):
    return ''.join(str(cell) for row in state for cell in row)

# A* 알고리즘
def a_star(start_state):
    visited = set()
    queue = []
    heapq.heappush(queue, (heuristic(start_state), 0, start_state, []))  # (f, g, state, path)

    while queue:
        f, g, current, path = heapq.heappop(queue)
        state_str = state_to_str(current)

        if state_str in visited:
            continue
        visited.add(state_str)

        if current == goal_state:
            return path + [current]

        for neighbor in get_neighbors(current):
            heapq.heappush(queue, (g + 1 + heuristic(neighbor), g + 1, neighbor, path + [current]))

    return None

# 초기 퍼즐 만들기
def generate_start_state():
    nums = list(range(10))
    random.shuffle(nums)
    return [nums[:5], nums[5:]]

# 퍼즐 출력 함수
def print_puzzle(state):
    for row in state:
        print(' '.join(str(x).rjust(2) for x in row))
    print()

# 실행
start_state = generate_start_state()
print("초기 퍼즐 상태:")
print_puzzle(start_state)

result = a_star(start_state)

if result:
    print("퍼즐 해결 경로 (총 %d 단계):" % (len(result) - 1))
    for step in result:
        print_puzzle(step)
else:
    print("해결 불가한 퍼즐입니다.")
