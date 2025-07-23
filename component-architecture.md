# Pomodoro Timer App - Component Architecture & Specifications

## 1. Architecture Overview

### 1.1 System Architecture Pattern
**Layered Architecture with Clean Architecture Principles**
- **Presentation Layer**: React Native components and screens
- **Business Logic Layer**: Services and state management
- **Data Access Layer**: Database operations and local storage
- **Infrastructure Layer**: Device APIs and external integrations

### 1.2 Technology Stack
- **Framework**: React Native with TypeScript
- **State Management**: Redux Toolkit with RTK Query
- **Database**: SQLite with react-native-sqlite-storage
- **Navigation**: React Navigation 6
- **Animations**: React Native Reanimated 3
- **UI Components**: Custom components with styled-components
- **Notifications**: @react-native-async-storage/async-storage
- **Testing**: Jest + React Native Testing Library

### 1.3 Project Structure
```
src/
├── components/           # Reusable UI components
│   ├── ui/              # Basic UI elements
│   ├── timer/           # Timer-specific components
│   ├── tasks/           # Task management components
│   ├── collection/      # Bird collection components
│   └── analytics/       # Analytics and charts
├── screens/             # Screen components
├── services/            # Business logic and API layer
├── store/               # Redux store configuration
├── models/              # TypeScript type definitions
├── utils/               # Utility functions
├── assets/              # Images, fonts, animations
├── hooks/               # Custom React hooks
└── constants/           # App constants and configurations
```

## 2. Core Component Specifications

### 2.1 Timer Components

#### 2.1.1 TimerScreen Component
```typescript
interface TimerScreenProps {
  navigation: NavigationProp<RootStackParamList, 'Timer'>;
}

interface TimerScreenState {
  currentSession: TimerSession | null;
  selectedTask: Task | null;
  timerDuration: number; // minutes
  isRunning: boolean;
  timeRemaining: number; // seconds
  showTimeSlider: boolean;
}

// Main screen component managing timer state and UI coordination
export const TimerScreen: React.FC<TimerScreenProps> = ({ navigation }) => {
  // State management and timer logic
  // Render CircularProgress, TimeSlider, ControlButtons, EggAnimation
};
```

#### 2.1.2 CircularProgress Component
```typescript
interface CircularProgressProps {
  progress: number; // 0-1 range
  size: number;
  strokeWidth: number;
  color: string;
  backgroundColor: string;
  children?: React.ReactNode;
}

// Animated circular progress bar with customizable styling
export const CircularProgress: React.FC<CircularProgressProps> = ({
  progress,
  size,
  strokeWidth,
  color,
  backgroundColor,
  children
}) => {
  // SVG-based circular progress with smooth animations
  // Uses React Native Reanimated for 60fps performance
};
```

#### 2.1.3 TimeSlider Component
```typescript
interface TimeSliderProps {
  value: number; // current value in minutes
  minimumValue: number; // minimum time (1 minute)
  maximumValue: number; // maximum time (180 minutes)
  onValueChange: (value: number) => void;
  onSlidingComplete: (value: number) => void;
  disabled: boolean;
}

// Custom slider for time selection with haptic feedback
export const TimeSlider: React.FC<TimeSliderProps> = ({
  value,
  minimumValue,
  maximumValue,
  onValueChange,
  onSlidingComplete,
  disabled
}) => {
  // Custom slider implementation with time labels
  // Haptic feedback on value changes
  // Visual indicators for common durations (25min, 50min, etc.)
};
```

#### 2.1.4 EggAnimation Component
```typescript
interface EggAnimationProps {
  progress: number; // 0-1, timer progress
  isActive: boolean; // whether timer is running
  onHatchComplete: (bird: Bird) => void;
  eggType: 'normal' | 'rare' | 'legendary';
}

// Animated egg that progressively hatches during timer sessions
export const EggAnimation: React.FC<EggAnimationProps> = ({
  progress,
  isActive,
  onHatchComplete,
  eggType
}) => {
  // Lottie animations for egg stages
  // Progressive crack appearances based on timer progress
  // Hatch reveal animation with bird emergence
};
```

#### 2.1.5 ControlButtons Component
```typescript
interface ControlButtonsProps {
  timerState: 'idle' | 'running' | 'paused';
  onPlay: () => void;
  onPause: () => void;
  onBreak: () => void;
  onEndSession: () => void;
  disabled: boolean;
}

// Timer control buttons with contextual display
export const ControlButtons: React.FC<ControlButtonsProps> = ({
  timerState,
  onPlay,
  onPause,
  onBreak,
  onEndSession,
  disabled
}) => {
  // Dynamic button layout based on timer state
  // Icon animations and haptic feedback
  // Confirmation dialogs for destructive actions
};
```

### 2.2 Task Management Components

#### 2.2.1 TaskBoard Component
```typescript
interface TaskBoardProps {
  tasks: Task[];
  selectedDate: Date;
  onTaskCreate: (task: Partial<Task>) => void;
  onTaskUpdate: (taskId: string, updates: Partial<Task>) => void;
  onTaskDelete: (taskId: string) => void;
}

// Kanban-style board with three columns
export const TaskBoard: React.FC<TaskBoardProps> = ({
  tasks,
  selectedDate,
  onTaskCreate,
  onTaskUpdate,
  onTaskDelete
}) => {
  // Three collapsible sections: Todo, In Progress, Done
  // Drag and drop functionality for status changes
  // Add task button and quick create modal
};
```

#### 2.2.2 TaskCard Component
```typescript
interface TaskCardProps {
  task: Task;
  onPress: () => void;
  onStatusChange: (status: TaskStatus) => void;
  onEdit: () => void;
  onDelete: () => void;
  draggable?: boolean;
}

// Individual task card with actions and metadata
export const TaskCard: React.FC<TaskCardProps> = ({
  task,
  onPress,
  onStatusChange,
  onEdit,
  onDelete,
  draggable = false
}) => {
  // Task title, description, tags, and time tracking
  // Swipe actions for quick operations
  // Visual indicators for priority and completion
};
```

#### 2.2.3 TaskSelector Component
```typescript
interface TaskSelectorProps {
  tasks: Task[];
  selectedTaskId: string | null;
  onTaskSelect: (taskId: string | null) => void;
  onCreateTask: () => void;
}

// Dropdown selector for timer task assignment
export const TaskSelector: React.FC<TaskSelectorProps> = ({
  tasks,
  selectedTaskId,
  onTaskSelect,
  onCreateTask
}) => {
  // Searchable dropdown with task filtering
  // Quick create option for new tasks
  // Recently used tasks prioritization
};
```

#### 2.2.4 TagManager Component
```typescript
interface TagManagerProps {
  tags: Tag[];
  selectedTags: string[];
  onTagToggle: (tagId: string) => void;
  onTagCreate: (tag: Partial<Tag>) => void;
  onTagEdit: (tagId: string, updates: Partial<Tag>) => void;
  onTagDelete: (tagId: string) => void;
}

// Tag creation and assignment interface
export const TagManager: React.FC<TagManagerProps> = ({
  tags,
  selectedTags,
  onTagToggle,
  onTagCreate,
  onTagEdit,
  onTagDelete
}) => {
  // Color-coded tag chips with selection state
  // Custom tag creation with color picker
  // Predefined tag suggestions
};
```

### 2.3 Collection Components

#### 2.3.1 BirdCollection Component
```typescript
interface BirdCollectionProps {
  birds: Bird[];
  unlockedBirds: UserBird[];
  onBirdPress: (bird: Bird) => void;
}

// Main collection display with grid layout
export const BirdCollection: React.FC<BirdCollectionProps> = ({
  birds,
  unlockedBirds,
  onBirdPress
}) => {
  // 2x4 grid layout for 8 birds
  // Locked/unlocked visual states
  // Rarity indicators and collection progress
};
```

#### 2.3.2 BirdCard Component
```typescript
interface BirdCardProps {
  bird: Bird;
  isUnlocked: boolean;
  unlockDate?: Date;
  onPress: () => void;
}

// Individual bird display card
export const BirdCard: React.FC<BirdCardProps> = ({
  bird,
  isUnlocked,
  unlockDate,
  onPress
}) => {
  // Bird image with unlock overlay
  // Rarity border styling
  // Unlock animation effects
};
```

#### 2.3.3 BirdReveal Component
```typescript
interface BirdRevealProps {
  bird: Bird;
  visible: boolean;
  onClose: () => void;
}

// Modal for new bird unlock celebration
export const BirdReveal: React.FC<BirdRevealProps> = ({
  bird,
  visible,
  onClose
}) => {
  // Celebratory animation sequence
  // Bird details and collection progress
  // Social sharing options
};
```

### 2.4 Analytics Components

#### 2.4.1 AnalyticsScreen Component
```typescript
interface AnalyticsScreenProps {
  navigation: NavigationProp<RootStackParamList, 'Analytics'>;
}

// Main analytics screen with multiple chart views
export const AnalyticsScreen: React.FC<AnalyticsScreenProps> = ({ navigation }) => {
  // Time period selector
  // Multiple chart types
  // Data export functionality
};
```

#### 2.4.2 TimeChart Component
```typescript
interface TimeChartProps {
  data: ChartDataPoint[];
  chartType: 'bar' | 'line' | 'area';
  timeRange: 'week' | 'month' | 'year' | 'all';
  height: number;
}

// Customizable time-based charts
export const TimeChart: React.FC<TimeChartProps> = ({
  data,
  chartType,
  timeRange,
  height
}) => {
  // Victory charts for data visualization
  // Interactive tooltips and zoom
  // Responsive design for different screen sizes
};
```

#### 2.4.3 TagAnalytics Component
```typescript
interface TagAnalyticsProps {
  tagData: TagStatistic[];
  displayType: 'pie' | 'bar' | 'list';
}

// Tag-based time distribution visualization
export const TagAnalytics: React.FC<TagAnalyticsProps> = ({
  tagData,
  displayType
}) => {
  // Pie chart for tag distribution
  // Bar chart for time comparisons
  // List view with detailed statistics
};
```

#### 2.4.4 SessionHistory Component
```typescript
interface SessionHistoryProps {
  sessions: TimerSession[];
  onSessionPress: (session: TimerSession) => void;
  groupBy: 'day' | 'week' | 'month';
}

// Chronological session history with grouping
export const SessionHistory: React.FC<SessionHistoryProps> = ({
  sessions,
  onSessionPress,
  groupBy
}) => {
  // Grouped session lists
  // Session details and statistics
  // Export and sharing functionality
};
```

## 3. Service Layer Architecture

### 3.1 TimerService
```typescript
class TimerService {
  private timer: NodeJS.Timeout | null = null;
  private currentSession: TimerSession | null = null;
  
  // Start a new timer session
  async startSession(taskId: string | null, duration: number): Promise<TimerSession>;
  
  // Pause/resume current session
  pauseSession(): void;
  resumeSession(): void;
  
  // End session with completion logic
  async endSession(completed: boolean): Promise<void>;
  
  // Get current session state
  getCurrentSession(): TimerSession | null;
  
  // Timer event callbacks
  onTick: (remainingTime: number) => void;
  onComplete: (session: TimerSession) => void;
  onBirdUnlock: (bird: Bird) => void;
}
```

### 3.2 DatabaseService
```typescript
class DatabaseService {
  // Database initialization and migrations
  async initialize(): Promise<void>;
  
  // Task operations
  async getTasks(userId: string, status?: TaskStatus): Promise<Task[]>;
  async createTask(task: Partial<Task>): Promise<Task>;
  async updateTask(taskId: string, updates: Partial<Task>): Promise<Task>;
  async deleteTask(taskId: string): Promise<void>;
  
  // Session operations
  async createSession(session: Partial<TimerSession>): Promise<TimerSession>;
  async updateSession(sessionId: string, updates: Partial<TimerSession>): Promise<TimerSession>;
  async getSessions(userId: string, dateRange?: DateRange): Promise<TimerSession[]>;
  
  // Bird operations
  async getBirds(): Promise<Bird[]>;
  async getUserBirds(userId: string): Promise<UserBird[]>;
  async unlockBird(userId: string, birdId: string, sessionId: string): Promise<UserBird>;
  
  // Analytics operations
  async getTagStatistics(userId: string, dateRange?: DateRange): Promise<TagStatistic[]>;
  async getDailyStats(userId: string, dateRange: DateRange): Promise<DailyStatistic[]>;
}
```

### 3.3 NotificationService
```typescript
class NotificationService {
  // Initialize notification permissions
  async initialize(): Promise<boolean>;
  
  // Schedule timer completion notification
  async scheduleTimerNotification(duration: number, taskTitle?: string): Promise<void>;
  
  // Cancel pending notifications
  async cancelNotifications(): Promise<void>;
  
  // Handle notification tap events
  onNotificationTap: (data: any) => void;
  
  // Check and request permissions
  async checkPermissions(): Promise<boolean>;
  async requestPermissions(): Promise<boolean>;
}
```

### 3.4 AnalyticsService
```typescript
class AnalyticsService {
  // Calculate tag-based statistics
  async calculateTagStats(userId: string, dateRange: DateRange): Promise<TagStatistic[]>;
  
  // Generate daily productivity data
  async getDailyProductivity(userId: string, dateRange: DateRange): Promise<ProductivityData[]>;
  
  // Calculate streaks and achievements
  async getStreakData(userId: string): Promise<StreakData>;
  
  // Export data for backup/sharing
  async exportData(userId: string, format: 'json' | 'csv'): Promise<string>;
  
  // Generate insights and recommendations
  async generateInsights(userId: string): Promise<ProductivityInsight[]>;
}
```

## 4. State Management Architecture

### 4.1 Redux Store Structure
```typescript
interface RootState {
  auth: AuthState;
  timer: TimerState;
  tasks: TasksState;
  collection: CollectionState;
  analytics: AnalyticsState;
  settings: SettingsState;
  ui: UIState;
}

// Timer slice
interface TimerState {
  currentSession: TimerSession | null;
  isRunning: boolean;
  timeRemaining: number;
  selectedTask: Task | null;
  timerDuration: number;
  eggProgress: number;
}

// Tasks slice
interface TasksState {
  tasks: Task[];
  tags: Tag[];
  selectedDate: Date;
  loading: boolean;
  error: string | null;
}

// Collection slice
interface CollectionState {
  birds: Bird[];
  userBirds: UserBird[];
  newlyUnlocked: Bird | null;
  showReveal: boolean;
}
```

### 4.2 Redux Toolkit Slices
```typescript
// Timer slice with actions and reducers
export const timerSlice = createSlice({
  name: 'timer',
  initialState,
  reducers: {
    startTimer: (state, action: PayloadAction<StartTimerPayload>) => {
      // Timer start logic
    },
    tickTimer: (state) => {
      // Decrement time remaining
    },
    pauseTimer: (state) => {
      // Pause timer state
    },
    completeSession: (state, action: PayloadAction<SessionCompletePayload>) => {
      // Session completion logic
    }
  },
  extraReducers: (builder) => {
    // Async thunk handlers
  }
});

// Async thunks for complex operations
export const startTimerSession = createAsyncThunk(
  'timer/startSession',
  async (params: StartSessionParams, { dispatch, getState }) => {
    // Complex session start logic with database operations
  }
);
```

## 5. Custom Hooks

### 5.1 Timer Hooks
```typescript
// Custom hook for timer management
export const useTimer = () => {
  const dispatch = useAppDispatch();
  const timerState = useAppSelector(selectTimerState);
  
  const startTimer = useCallback((taskId: string | null, duration: number) => {
    dispatch(startTimerSession({ taskId, duration }));
  }, [dispatch]);
  
  const pauseTimer = useCallback(() => {
    dispatch(pauseTimerAction());
  }, [dispatch]);
  
  return {
    ...timerState,
    startTimer,
    pauseTimer,
    resumeTimer,
    endTimer
  };
};

// Hook for background timer persistence
export const useTimerPersistence = () => {
  useEffect(() => {
    // Handle app state changes
    const handleAppStateChange = (nextAppState: string) => {
      if (nextAppState === 'background') {
        // Save timer state
      } else if (nextAppState === 'active') {
        // Restore timer state
      }
    };
    
    AppState.addEventListener('change', handleAppStateChange);
    return () => AppState.removeEventListener('change', handleAppStateChange);
  }, []);
};
```

### 5.2 Data Hooks
```typescript
// Hook for task management
export const useTasks = (date?: Date) => {
  const [tasks, setTasks] = useState<Task[]>([]);
  const [loading, setLoading] = useState(false);
  
  const createTask = useCallback(async (taskData: Partial<Task>) => {
    // Task creation logic
  }, []);
  
  const updateTask = useCallback(async (taskId: string, updates: Partial<Task>) => {
    // Task update logic
  }, []);
  
  return {
    tasks,
    loading,
    createTask,
    updateTask,
    deleteTask,
    refreshTasks
  };
};

// Hook for analytics data
export const useAnalytics = (timeRange: TimeRange) => {
  const [analyticsData, setAnalyticsData] = useState<AnalyticsData | null>(null);
  
  useEffect(() => {
    const loadAnalytics = async () => {
      // Load analytics data based on time range
    };
    
    loadAnalytics();
  }, [timeRange]);
  
  return {
    analyticsData,
    refreshAnalytics,
    exportData
  };
};
```

## 6. Animation Architecture

### 6.1 Animation Libraries
- **React Native Reanimated 3**: High-performance animations
- **Lottie React Native**: Complex egg hatching animations
- **React Native Gesture Handler**: Smooth gesture interactions

### 6.2 Animation Components
```typescript
// Reusable animation hooks
export const useSharedTransition = (value: number, config?: TimingConfig) => {
  return useSharedValue(withTiming(value, config));
};

export const useRotationAnimation = (rotating: boolean) => {
  const rotation = useSharedValue(0);
  
  useEffect(() => {
    rotation.value = withRepeat(
      withTiming(rotating ? 360 : 0, { duration: 1000 }),
      -1,
      false
    );
  }, [rotating]);
  
  return useAnimatedStyle(() => ({
    transform: [{ rotate: `${rotation.value}deg` }]
  }));
};
```

## 7. Error Handling and Logging

### 7.1 Error Boundaries
```typescript
class ErrorBoundary extends React.Component<Props, State> {
  constructor(props: Props) {
    super(props);
    this.state = { hasError: false, error: null };
  }
  
  static getDerivedStateFromError(error: Error): State {
    return { hasError: true, error };
  }
  
  componentDidCatch(error: Error, errorInfo: ErrorInfo) {
    // Log error to crash reporting service
    console.error('Error caught by boundary:', error, errorInfo);
  }
  
  render() {
    if (this.state.hasError) {
      return <ErrorFallback error={this.state.error} />;
    }
    
    return this.props.children;
  }
}
```

### 7.2 Logging Service
```typescript
class LoggingService {
  static info(message: string, data?: any): void {
    console.log(`[INFO] ${message}`, data);
  }
  
  static error(message: string, error?: Error): void {
    console.error(`[ERROR] ${message}`, error);
  }
  
  static timer(action: string, data?: any): void {
    console.log(`[TIMER] ${action}`, data);
  }
}
```

## 8. Testing Architecture

### 8.1 Component Testing
```typescript
// Example test for TimerScreen
describe('TimerScreen', () => {
  it('should start timer when play button is pressed', async () => {
    const mockStartTimer = jest.fn();
    render(<TimerScreen />);
    
    const playButton = screen.getByTestId('play-button');
    fireEvent.press(playButton);
    
    expect(mockStartTimer).toHaveBeenCalled();
  });
  
  it('should display egg animation when timer is running', () => {
    render(<TimerScreen />);
    // Test egg animation presence
  });
});
```

### 8.2 Service Testing
```typescript
// Example test for TimerService
describe('TimerService', () => {
  let timerService: TimerService;
  
  beforeEach(() => {
    timerService = new TimerService();
  });
  
  it('should create session when starting timer', async () => {
    const session = await timerService.startSession('task-1', 25);
    expect(session).toBeDefined();
    expect(session.plannedDuration).toBe(25);
  });
});
```

## 9. Performance Optimization

### 9.1 Component Optimization
- **React.memo**: Prevent unnecessary re-renders
- **useMemo/useCallback**: Memoize expensive calculations
- **FlatList**: Efficient list rendering for large datasets
- **Image Optimization**: Proper sizing and caching

### 9.2 Bundle Optimization
- **Code Splitting**: Lazy load non-critical components
- **Tree Shaking**: Remove unused code
- **Asset Optimization**: Compress images and animations
- **Metro Configuration**: Optimize bundling process

## 10. Platform-Specific Considerations

### 10.1 iOS Specific
- **Background App Refresh**: Timer persistence when backgrounded
- **Push Notifications**: Proper notification scheduling
- **Haptic Feedback**: Native haptic integration
- **App Store Guidelines**: Compliance with review guidelines

### 10.2 Android Specific
- **Background Services**: Foreground service for timer
- **Notification Channels**: Proper notification categorization
- **Battery Optimization**: Whitelist app from battery optimization
- **Material Design**: Native Android UI patterns

This component architecture provides a comprehensive foundation for building a scalable, maintainable, and performant Pomodoro Timer App with all the specified features.