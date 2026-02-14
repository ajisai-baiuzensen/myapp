import React, { useState, useEffect, useRef } from 'react';
import { Camera, Map, BookOpen, Archive, Share2, Plus, X, Check, MapPin, Navigation, Circle } from 'lucide-react';

// 江戸時代の古地図風カラーパレット（日本画の色彩）
const colors = {
  墨色: '#3D3D3D',
  古紙: '#E8DCC8',
  渋茶: '#8B6F47',
  褪朱: '#C17A6F',
  藍鼠: '#5A6F7F',
  萌黄: '#7FA563',
  薄墨: '#9A8F82',
  黄土: '#C9A961',
  白群: '#B8C5D0',
};

export default function TravelJournalApp() {
  const [currentPage, setCurrentPage] = useState('home');
  const [travels, setTravels] = useState([]);
  const [diaries, setDiaries] = useState([]);
  const [albums, setAlbums] = useState([]);
  const [isTracking, setIsTracking] = useState(false);
  const [currentRoute, setCurrentRoute] = useState([]);
  const [showSaveModal, setShowSaveModal] = useState(false);
  const [travelTitle, setTravelTitle] = useState('');
  const [diaryTitle, setDiaryTitle] = useState('');
  const [diaryContent, setDiaryContent] = useState('');
  const [albumTitle, setAlbumTitle] = useState('');
  const [selectedDiary, setSelectedDiary] = useState(null);
  const [selectedAlbum, setSelectedAlbum] = useState(null);
  const [selectedPhotos, setSelectedPhotos] = useState([]);
  const [showUserModal, setShowUserModal] = useState(false);
  const [userProfile, setUserProfile] = useState({
    name: '',
    age: '',
    gender: '',
    transportation: [],
    vehicle: '',
    camera: '',
    recommendations: ''
  });
  const watchId = useRef(null);

  // ストレージからデータを読み込み
  useEffect(() => {
    loadData();
  }, []);

  const loadData = async () => {
    try {
      const travelsResult = await window.storage.get('travels');
      const diariesResult = await window.storage.get('diaries');
      const albumsResult = await window.storage.get('albums');
      const userResult = await window.storage.get('userProfile');
      
      if (travelsResult) setTravels(JSON.parse(travelsResult.value));
      if (diariesResult) setDiaries(JSON.parse(diariesResult.value));
      if (albumsResult) setAlbums(JSON.parse(albumsResult.value));
      if (userResult) setUserProfile(JSON.parse(userResult.value));
    } catch (error) {
      console.log('初回起動またはデータなし');
    }
  };

  const saveTravels = async (newTravels) => {
    try {
      await window.storage.set('travels', JSON.stringify(newTravels));
      setTravels(newTravels);
    } catch (error) {
      console.error('保存エラー:', error);
    }
  };

  const saveDiaries = async (newDiaries) => {
    try {
      await window.storage.set('diaries', JSON.stringify(newDiaries));
      setDiaries(newDiaries);
    } catch (error) {
      console.error('保存エラー:', error);
    }
  };

  const saveAlbums = async (newAlbums) => {
    try {
      await window.storage.set('albums', JSON.stringify(newAlbums));
      setAlbums(newAlbums);
    } catch (error) {
      console.error('保存エラー:', error);
    }
  };

  const saveUserProfile = async () => {
    if (!userProfile.name.trim()) {
      alert('お名前を入力してください');
      return;
    }

    try {
      await window.storage.set('userProfile', JSON.stringify(userProfile));
      setShowUserModal(false);
      alert('プロフィールを保存しました');
    } catch (error) {
      console.error('保存エラー:', error);
      alert('保存に失敗しました');
    }
  };

  const handleTransportationChange = (value) => {
    setUserProfile(prev => {
      const transportation = prev.transportation.includes(value)
        ? prev.transportation.filter(t => t !== value)
        : [...prev.transportation, value];
      return { ...prev, transportation };
    });
  };

  // 位置情報トラッキング
  const startTracking = () => {
    if ('geolocation' in navigator) {
      setIsTracking(true);
      setCurrentRoute([]);
      
      watchId.current = navigator.geolocation.watchPosition(
        (position) => {
          const point = {
            lat: position.coords.latitude,
            lng: position.coords.longitude,
            timestamp: Date.now()
          };
          setCurrentRoute(prev => [...prev, point]);
        },
        (error) => {
          console.error('位置情報エラー:', error);
          alert('位置情報の取得に失敗しました');
        },
        { enableHighAccuracy: true, maximumAge: 10000, timeout: 5000 }
      );
    } else {
      alert('お使いのブラウザは位置情報に対応していません');
    }
  };

  const stopTracking = () => {
    if (watchId.current) {
      navigator.geolocation.clearWatch(watchId.current);
      watchId.current = null;
    }
    setIsTracking(false);
    if (currentRoute.length > 0) {
      setShowSaveModal(true);
    }
  };

  const saveTravel = async () => {
    if (!travelTitle.trim()) {
      alert('旅のタイトルを入力してください');
      return;
    }

    const newTravel = {
      id: Date.now(),
      title: travelTitle,
      route: currentRoute,
      diary: selectedDiary,
      album: selectedAlbum,
      userName: userProfile.name || '旅人',
      createdAt: new Date().toISOString()
    };

    const newTravels = [...travels, newTravel];
    await saveTravels(newTravels);
    
    setShowSaveModal(false);
    setTravelTitle('');
    setSelectedDiary(null);
    setSelectedAlbum(null);
    setCurrentRoute([]);
    setCurrentPage('records');
  };

  const saveDiary = async () => {
    if (!diaryTitle.trim() || !diaryContent.trim()) {
      alert('タイトルと本文を入力してください');
      return;
    }

    const newDiary = {
      id: Date.now(),
      title: diaryTitle,
      content: diaryContent,
      createdAt: new Date().toISOString()
    };

    const newDiaries = [...diaries, newDiary];
    await saveDiaries(newDiaries);
    
    setDiaryTitle('');
    setDiaryContent('');
    alert('日記を保存しました');
  };

  const saveAlbum = async () => {
    if (!albumTitle.trim()) {
      alert('アルバムタイトルを入力してください');
      return;
    }

    const newAlbum = {
      id: Date.now(),
      title: albumTitle,
      photos: selectedPhotos,
      createdAt: new Date().toISOString()
    };

    const newAlbums = [...albums, newAlbum];
    await saveAlbums(newAlbums);
    
    setAlbumTitle('');
    setSelectedPhotos([]);
    alert('アルバムを保存しました');
  };

  const handlePhotoUpload = (e) => {
    const files = Array.from(e.target.files);
    const readers = files.map(file => {
      return new Promise((resolve) => {
        const reader = new FileReader();
        reader.onload = (e) => resolve(e.target.result);
        reader.readAsDataURL(file);
      });
    });

    Promise.all(readers).then(photos => {
      setSelectedPhotos(prev => [...prev, ...photos]);
    });
  };

  const shareTravel = (travel) => {
    const shareText = `${travel.title}\n\n旅の記録をシェアします！`;
    
    if (navigator.share) {
      navigator.share({
        title: travel.title,
        text: shareText,
      }).catch(err => console.log('シェアエラー:', err));
    } else {
      alert('この機能はお使いのブラウザでは利用できません');
    }
  };

  // ホーム画面
  const HomePage = () => (
    <div className="min-h-screen flex flex-col items-center justify-center p-8 relative overflow-hidden" style={{ 
      backgroundColor: '#F5E6D3',
      backgroundImage: 'radial-gradient(circle, rgba(232, 153, 102, 0.08) 1px, transparent 1px)',
      backgroundSize: '30px 30px'
    }}>
      {/* 柔らかな装飾要素 */}
      <div className="absolute inset-0 opacity-[0.04]" style={{
        backgroundImage: `radial-gradient(circle at 20% 30%, ${colors.褪朱} 0%, transparent 50%),
                         radial-gradient(circle at 80% 70%, ${colors.黄土} 0%, transparent 50%)`
      }} />

      {/* 装飾的な枠線 */}
      <div className="absolute inset-8 border-4 rounded-lg opacity-20" style={{ borderColor: colors.渋茶 }} />
      <div className="absolute inset-12 border-2 rounded-lg opacity-10" style={{ borderColor: colors.渋茶 }} />

      <div className="text-center mb-16 relative z-10">
        <h1 className="font-bold mb-4 relative inline-block" style={{ 
          fontSize: '4rem',
          color: colors.墨色,
          fontFamily: "'Noto Serif JP', serif",
          textShadow: '2px 2px 0px rgba(139, 111, 71, 0.2)',
          letterSpacing: '0.1em'
        }}>
          日本旅行記録帳
        </h1>
      </div>

      <div className="grid grid-cols-2 gap-6 max-w-2xl w-full relative z-10">
        {[
          { id: 'map', icon: Map, label: '地図', color: colors.藍鼠, emoji: '🗾' },
          { id: 'diary', icon: BookOpen, label: '日記', color: colors.褪朱, emoji: '📜' },
          { id: 'photos', icon: Camera, label: '写真', color: colors.黄土, emoji: '🎴' },
          { id: 'records', icon: Archive, label: '旅の記録', color: colors.萌黄, emoji: '📚' },
        ].map((item) => (
          <button
            key={item.id}
            onClick={() => setCurrentPage(item.id)}
            className="group relative overflow-hidden rounded-lg p-8 transition-all duration-300 active:scale-95 hover:scale-110 hover:shadow-2xl"
            style={{
              backgroundColor: colors.古紙,
              border: `3px solid ${item.color}`,
              boxShadow: '4px 4px 0px rgba(0,0,0,0.1)'
            }}
          >
            <div className="absolute top-2 right-2 text-3xl opacity-20 group-hover:opacity-30 transition-opacity">
              {item.emoji}
            </div>
            <item.icon className="w-12 h-12 mb-4 mx-auto" style={{ color: item.color }} />
            <div className="text-2xl font-bold" style={{ 
              color: colors.墨色,
              fontFamily: "'Noto Serif JP', serif"
            }}>
              {item.label}
            </div>
          </button>
        ))}
      </div>

      {/* 装飾波線（古地図風） */}
      <svg className="absolute bottom-0 left-0 w-full" style={{ height: '150px', opacity: 0.08 }} viewBox="0 0 1200 150">
        <path d="M0,80 Q300,20 600,80 T1200,80 L1200,150 L0,150 Z" fill={colors.藍鼠} />
      </svg>

      {/* ユーザー登録ボタン */}
      <div className="absolute bottom-12 left-1/2 transform -translate-x-1/2 z-10">
        <button
          onClick={() => setShowUserModal(true)}
          className="px-8 py-3 rounded-lg text-base font-bold transition-all duration-300 active:scale-95 hover:scale-110 hover:shadow-2xl"
          style={{
            backgroundColor: colors.渋茶,
            color: colors.古紙,
            fontFamily: "'Noto Serif JP', serif",
            boxShadow: '3px 3px 0px rgba(0,0,0,0.2)'
          }}
        >
          {userProfile.name ? `👤 ${userProfile.name}さんのプロフィール` : '📝 ユーザー登録'}
        </button>
      </div>
    </div>
  );

  // 地図ページ
  const MapPage = () => (
    <div className="min-h-screen p-8" style={{ backgroundColor: colors.古紙 }}>
      <button 
        onClick={() => setCurrentPage('home')}
        className="mb-6 px-6 py-3 rounded-lg font-bold transition-all duration-300 active:scale-95 hover:scale-110 hover:shadow-lg"
        style={{ 
          backgroundColor: colors.墨色,
          color: colors.古紙,
          fontFamily: "'Noto Serif JP', serif",
          boxShadow: '3px 3px 0px rgba(0,0,0,0.2)'
        }}
      >
        ← 戻る
      </button>

      <div className="max-w-4xl mx-auto">
        <h2 className="text-4xl font-bold mb-8 text-center" style={{ 
          color: colors.藍鼠,
          fontFamily: "'Noto Serif JP', serif",
          letterSpacing: '0.1em'
        }}>
          旅路の地図
        </h2>

        {/* 古地図風マップエリア */}
        <div className="rounded-lg p-8 mb-8 relative overflow-hidden" style={{ 
          backgroundColor: '#e5dcc5',
          border: `4px solid ${colors.渋茶}`,
          minHeight: '450px',
          boxShadow: 'inset 0 0 20px rgba(0,0,0,0.1), 4px 4px 0px rgba(0,0,0,0.15)'
        }}>
          {/* 古地図風テクスチャ */}
          <div className="absolute inset-0 opacity-[0.04]" style={{
            backgroundImage: `repeating-linear-gradient(45deg, transparent, transparent 10px, ${colors.墨色} 10px, ${colors.墨色} 11px)`
          }} />

          {/* 装飾的な方位記号 */}
          <div className="absolute top-4 right-4 w-16 h-16 rounded-full flex items-center justify-center" style={{
            border: `2px solid ${colors.渋茶}`,
            backgroundColor: colors.古紙,
            opacity: 0.7
          }}>
            <div className="text-center" style={{ color: colors.墨色, fontFamily: "'Noto Serif JP', serif", fontSize: '0.7rem' }}>
              <div>北</div>
            </div>
          </div>

          <div className="relative z-10 text-center flex flex-col items-center justify-center" style={{ minHeight: '400px' }}>
            {!isTracking && currentRoute.length === 0 && (
              <div className="py-8">
                <div className="mb-6 p-4 rounded-full inline-block" style={{ backgroundColor: 'rgba(90, 111, 127, 0.1)' }}>
                  <Map className="w-20 h-20" style={{ color: colors.藍鼠 }} />
                </div>
                <p className="text-2xl mb-2" style={{ 
                  color: colors.墨色,
                  fontFamily: "'Noto Serif JP', serif"
                }}>
                  旅の準備が整いました
                </p>
                <p className="text-base mb-8" style={{ color: colors.薄墨 }}>
                  記録を開始すると、移動経路が保存されます
                </p>
              </div>
            )}

            {isTracking && (
              <div className="py-8">
                <div className="relative mb-8">
                  <div className="animate-ping absolute inline-flex h-full w-full rounded-full opacity-75" style={{ backgroundColor: colors.褪朱 }}></div>
                  <div className="relative p-4 rounded-full inline-block" style={{ backgroundColor: 'rgba(193, 122, 111, 0.2)' }}>
                    <MapPin className="w-20 h-20" style={{ color: colors.褪朱 }} />
                  </div>
                </div>
                <p className="text-2xl mb-2 font-bold" style={{ 
                  color: colors.墨色,
                  fontFamily: "'Noto Serif JP', serif"
                }}>
                  旅の途中...
                </p>
                <p className="text-lg mb-2" style={{ color: colors.薄墨 }}>
                  記録地点: {currentRoute.length} ヶ所
                </p>
                <p className="text-sm" style={{ color: colors.薄墨 }}>
                  位置情報を記録しています
                </p>
              </div>
            )}

            {!isTracking && currentRoute.length > 0 && (
              <div className="py-8">
                <div className="mb-6 p-4 rounded-full inline-block" style={{ backgroundColor: 'rgba(127, 165, 99, 0.2)' }}>
                  <Check className="w-20 h-20" style={{ color: colors.萌黄 }} />
                </div>
                <p className="text-2xl mb-2 font-bold" style={{ 
                  color: colors.墨色,
                  fontFamily: "'Noto Serif JP', serif"
                }}>
                  旅の記録が完了しました
                </p>
                <p className="text-lg mb-8" style={{ color: colors.薄墨 }}>
                  記録地点: {currentRoute.length} ヶ所
                </p>
              </div>
            )}
          </div>
        </div>

        {/* コントロールボタン */}
        <div className="flex gap-4 justify-center">
          {!isTracking && currentRoute.length === 0 && (
            <button
              onClick={startTracking}
              className="px-10 py-4 rounded-lg text-lg font-bold transition-all duration-300 active:scale-95 hover:scale-110 hover:shadow-2xl"
              style={{
                backgroundColor: colors.褪朱,
                color: colors.古紙,
                fontFamily: "'Noto Serif JP', serif",
                boxShadow: '4px 4px 0px rgba(0,0,0,0.2)',
                border: `2px solid ${colors.墨色}20`
              }}
            >
              旅の記録を始める
            </button>
          )}

          {isTracking && (
            <button
              onClick={stopTracking}
              className="px-10 py-4 rounded-lg text-lg font-bold transition-all duration-300 active:scale-95 hover:scale-110 hover:shadow-2xl"
              style={{
                backgroundColor: colors.藍鼠,
                color: colors.古紙,
                fontFamily: "'Noto Serif JP', serif",
                boxShadow: '4px 4px 0px rgba(0,0,0,0.2)',
                border: `2px solid ${colors.墨色}20`
              }}
            >
              旅の記録を終了する
            </button>
          )}

          {!isTracking && currentRoute.length > 0 && (
            <button
              onClick={() => setShowSaveModal(true)}
              className="px-10 py-4 rounded-lg text-lg font-bold transition-all duration-300 active:scale-95 hover:scale-110 hover:shadow-2xl"
              style={{
                backgroundColor: colors.黄土,
                color: colors.古紙,
                fontFamily: "'Noto Serif JP', serif",
                boxShadow: '4px 4px 0px rgba(0,0,0,0.2)',
                border: `2px solid ${colors.墨色}20`
              }}
            >
              旅の記録を保存する
            </button>
          )}
        </div>
      </div>

      {/* 保存モーダル */}
      {showSaveModal && (
        <div className="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center p-4 z-50">
          <div className="rounded-lg p-8 max-w-2xl w-full max-h-[90vh] overflow-y-auto" style={{ 
            backgroundColor: colors.古紙,
            border: `4px solid ${colors.渋茶}`,
            boxShadow: '8px 8px 0px rgba(0,0,0,0.3)'
          }}>
            <div className="flex items-center justify-between mb-6">
              <h3 className="text-3xl font-bold" style={{ 
                color: colors.墨色,
                fontFamily: "'Noto Serif JP', serif"
              }}>
                旅を保存
              </h3>
              <button 
                onClick={() => setShowSaveModal(false)}
                className="transition-all duration-200 active:scale-90 hover:scale-110 hover:shadow-lg"
              >
                <X className="w-8 h-8" style={{ color: colors.薄墨 }} />
              </button>
            </div>

            <div className="space-y-6">
              <div>
                <label className="block mb-2 font-bold" style={{ color: colors.墨色 }}>
                  旅のタイトル *
                </label>
                <input
                  type="text"
                  value={travelTitle}
                  onChange={(e) => setTravelTitle(e.target.value)}
                  placeholder="例: 京都への旅"
                  className="w-full px-4 py-3 rounded-lg border-2 focus:outline-none transition-all"
                  style={{ 
                    borderColor: colors.藍鼠,
                    fontFamily: "'Noto Serif JP', serif",
                    backgroundColor: '#f8f5eb'
                  }}
                />
              </div>

              <div>
                <label className="block mb-2 font-bold" style={{ color: colors.墨色 }}>
                  日記を追加（任意）
                </label>
                <select
                  value={selectedDiary?.id || ''}
                  onChange={(e) => {
                    const diary = diaries.find(d => d.id === Number(e.target.value));
                    setSelectedDiary(diary);
                  }}
                  className="w-full px-4 py-3 rounded-lg border-2 focus:outline-none transition-all"
                  style={{ 
                    borderColor: colors.褪朱,
                    fontFamily: "'Noto Serif JP', serif",
                    backgroundColor: '#f8f5eb'
                  }}
                >
                  <option value="">選択してください</option>
                  {diaries.map(diary => (
                    <option key={diary.id} value={diary.id}>{diary.title}</option>
                  ))}
                </select>
              </div>

              <div>
                <label className="block mb-2 font-bold" style={{ color: colors.墨色 }}>
                  アルバムを追加（任意）
                </label>
                <select
                  value={selectedAlbum?.id || ''}
                  onChange={(e) => {
                    const album = albums.find(a => a.id === Number(e.target.value));
                    setSelectedAlbum(album);
                  }}
                  className="w-full px-4 py-3 rounded-lg border-2 focus:outline-none transition-all"
                  style={{ 
                    borderColor: colors.黄土,
                    fontFamily: "'Noto Serif JP', serif",
                    backgroundColor: '#f8f5eb'
                  }}
                >
                  <option value="">選択してください</option>
                  {albums.map(album => (
                    <option key={album.id} value={album.id}>{album.title}</option>
                  ))}
                </select>
              </div>

              <button
                onClick={saveTravel}
                className="w-full px-8 py-4 rounded-lg text-xl font-bold transition-all duration-300 active:scale-95 hover:scale-110 hover:shadow-lg"
                style={{
                  backgroundColor: colors.萌黄,
                  color: colors.古紙,
                  fontFamily: "'Noto Serif JP', serif",
                  boxShadow: '4px 4px 0px rgba(0,0,0,0.2)'
                }}
              >
                保存する
              </button>
            </div>
          </div>
        </div>
      )}
    </div>
  );

  // 日記ページ
  const DiaryPage = () => (
    <div className="min-h-screen p-8" style={{ backgroundColor: colors.古紙 }}>
      <button 
        onClick={() => setCurrentPage('home')}
        className="mb-6 px-6 py-3 rounded-lg font-bold transition-all duration-300 active:scale-95 hover:scale-110 hover:shadow-lg"
        style={{ 
          backgroundColor: colors.墨色,
          color: colors.古紙,
          fontFamily: "'Noto Serif JP', serif",
          boxShadow: '3px 3px 0px rgba(0,0,0,0.2)'
        }}
      >
        ← 戻る
      </button>

      <div className="max-w-4xl mx-auto">
        <h2 className="text-4xl font-bold mb-8 text-center" style={{ 
          color: colors.褪朱,
          fontFamily: "'Noto Serif JP', serif",
          letterSpacing: '0.1em'
        }}>
          旅の日記
        </h2>

        {/* 新規日記作成 */}
        <div className="rounded-lg p-8 mb-8" style={{ 
          backgroundColor: '#e8dcc8',
          border: `4px solid ${colors.褪朱}`,
          boxShadow: '4px 4px 0px rgba(0,0,0,0.15)'
        }}>
          <h3 className="text-2xl font-bold mb-6" style={{ 
            color: colors.墨色,
            fontFamily: "'Noto Serif JP', serif"
          }}>
            ✍️ 新しい日記
          </h3>

          <div className="space-y-4">
            <input
              type="text"
              value={diaryTitle}
              onChange={(e) => setDiaryTitle(e.target.value)}
              placeholder="タイトル"
              className="w-full px-4 py-3 rounded-lg border-2 focus:outline-none text-xl transition-all"
              style={{ 
                borderColor: colors.褪朱,
                fontFamily: "'Noto Serif JP', serif",
                backgroundColor: '#f8f5eb'
              }}
            />
            <textarea
              value={diaryContent}
              onChange={(e) => setDiaryContent(e.target.value)}
              placeholder="今日の旅の思い出を書きましょう..."
              rows="8"
              className="w-full px-4 py-3 rounded-lg border-2 focus:outline-none resize-none transition-all"
              style={{ 
                borderColor: colors.褪朱,
                fontFamily: "'Noto Serif JP', serif",
                lineHeight: '1.8',
                backgroundColor: '#f8f5eb'
              }}
            />
            <button
              onClick={saveDiary}
              className="w-full px-8 py-4 rounded-lg text-xl font-bold transition-all duration-300 active:scale-95 hover:scale-110 hover:shadow-lg"
              style={{
                backgroundColor: colors.褪朱,
                color: colors.古紙,
                fontFamily: "'Noto Serif JP', serif",
                boxShadow: '4px 4px 0px rgba(0,0,0,0.2)'
              }}
            >
              保存する
            </button>
          </div>
        </div>

        {/* 過去の日記一覧 */}
        <h3 className="text-2xl font-bold mb-6" style={{ 
          color: colors.墨色,
          fontFamily: "'Noto Serif JP', serif"
        }}>
          📚 過去の日記
        </h3>

        <div className="grid gap-4">
          {diaries.length === 0 ? (
            <div className="text-center py-16 rounded-lg" style={{ 
              backgroundColor: '#e8dcc8',
              border: `2px dashed ${colors.薄墨}`
            }}>
              <BookOpen className="w-16 h-16 mx-auto mb-4" style={{ color: colors.薄墨 }} />
              <p style={{ color: colors.薄墨, fontFamily: "'Noto Serif JP', serif" }}>
                まだ日記がありません
              </p>
            </div>
          ) : (
            diaries.slice().reverse().map(diary => (
              <div
                key={diary.id}
                className="rounded-lg p-6 transition-all duration-200 hover:shadow-lg cursor-pointer active:scale-98"
                style={{ 
                  backgroundColor: '#e8dcc8',
                  border: `2px solid ${colors.褪朱}`,
                  boxShadow: '3px 3px 0px rgba(0,0,0,0.1)'
                }}
              >
                <h4 className="text-xl font-bold mb-2" style={{ 
                  color: colors.墨色,
                  fontFamily: "'Noto Serif JP', serif"
                }}>
                  {diary.title}
                </h4>
                <p className="mb-3" style={{ 
                  color: colors.薄墨,
                  fontFamily: "'Noto Serif JP', serif",
                  lineHeight: '1.8'
                }}>
                  {diary.content.substring(0, 100)}
                  {diary.content.length > 100 && '...'}
                </p>
                <div className="text-sm" style={{ color: colors.薄墨 }}>
                  {new Date(diary.createdAt).toLocaleDateString('ja-JP', {
                    year: 'numeric',
                    month: 'long',
                    day: 'numeric'
                  })}
                </div>
              </div>
            ))
          )}
        </div>
      </div>
    </div>
  );

  // 写真ページ
  const PhotosPage = () => (
    <div className="min-h-screen p-8" style={{ backgroundColor: colors.古紙 }}>
      <button 
        onClick={() => setCurrentPage('home')}
        className="mb-6 px-6 py-3 rounded-lg font-bold transition-all duration-300 active:scale-95 hover:scale-110 hover:shadow-lg"
        style={{ 
          backgroundColor: colors.墨色,
          color: colors.古紙,
          fontFamily: "'Noto Serif JP', serif",
          boxShadow: '3px 3px 0px rgba(0,0,0,0.2)'
        }}
      >
        ← 戻る
      </button>

      <div className="max-w-4xl mx-auto">
        <h2 className="text-4xl font-bold mb-8 text-center" style={{ 
          color: colors.黄土,
          fontFamily: "'Noto Serif JP', serif",
          letterSpacing: '0.1em'
        }}>
          旅の写真
        </h2>

        {/* 新規アルバム作成 */}
        <div className="rounded-lg p-8 mb-8" style={{ 
          backgroundColor: '#e8dcc8',
          border: `4px solid ${colors.黄土}`,
          boxShadow: '4px 4px 0px rgba(0,0,0,0.15)'
        }}>
          <h3 className="text-2xl font-bold mb-6" style={{ 
            color: colors.墨色,
            fontFamily: "'Noto Serif JP', serif"
          }}>
            📸 新しいアルバム
          </h3>

          <div className="space-y-4">
            <input
              type="text"
              value={albumTitle}
              onChange={(e) => setAlbumTitle(e.target.value)}
              placeholder="アルバムタイトル"
              className="w-full px-4 py-3 rounded-lg border-2 focus:outline-none text-xl transition-all"
              style={{ 
                borderColor: colors.黄土,
                fontFamily: "'Noto Serif JP', serif",
                backgroundColor: '#f8f5eb'
              }}
            />

            <label className="block">
              <div className="w-full px-8 py-12 rounded-lg border-2 border-dashed text-center cursor-pointer transition-all duration-200 hover:border-solid hover:shadow-lg active:scale-98"
                   style={{ borderColor: colors.黄土, backgroundColor: '#f8f5eb' }}>
                <Camera className="w-16 h-16 mx-auto mb-4" style={{ color: colors.黄土 }} />
                <p className="text-lg font-bold" style={{ 
                  color: colors.墨色,
                  fontFamily: "'Noto Serif JP', serif"
                }}>
                  写真を追加
                </p>
                <p className="text-sm mt-2" style={{ color: colors.薄墨 }}>
                  クリックして写真を選択
                </p>
              </div>
              <input
                type="file"
                accept="image/*"
                multiple
                onChange={handlePhotoUpload}
                className="hidden"
              />
            </label>

            {selectedPhotos.length > 0 && (
              <div>
                <p className="mb-3 font-bold" style={{ color: colors.墨色 }}>
                  選択中: {selectedPhotos.length}枚
                </p>
                <div className="grid grid-cols-3 gap-3 max-h-64 overflow-y-auto">
                  {selectedPhotos.map((photo, index) => (
                    <div key={index} className="relative aspect-square rounded-lg overflow-hidden border-2" style={{ borderColor: colors.黄土 }}>
                      <img src={photo} alt="" className="w-full h-full object-cover" />
                      <button
                        onClick={() => setSelectedPhotos(prev => prev.filter((_, i) => i !== index))}
                        className="absolute top-1 right-1 bg-black bg-opacity-50 rounded-full p-1 transition-all duration-200 active:scale-90"
                      >
                        <X className="w-4 h-4 text-white" />
                      </button>
                    </div>
                  ))}
                </div>
              </div>
            )}

            <button
              onClick={saveAlbum}
              disabled={selectedPhotos.length === 0}
              className="w-full px-8 py-4 rounded-lg text-xl font-bold transition-all duration-300 active:scale-95 hover:scale-110 disabled:opacity-50"
              style={{
                backgroundColor: colors.黄土,
                color: colors.古紙,
                fontFamily: "'Noto Serif JP', serif",
                boxShadow: '4px 4px 0px rgba(0,0,0,0.2)'
              }}
            >
              アルバムを保存
            </button>
          </div>
        </div>

        {/* 過去のアルバム一覧 */}
        <h3 className="text-2xl font-bold mb-6" style={{ 
          color: colors.墨色,
          fontFamily: "'Noto Serif JP', serif"
        }}>
          📚 アルバム一覧
        </h3>

        <div className="grid gap-4">
          {albums.length === 0 ? (
            <div className="text-center py-16 rounded-lg" style={{ 
              backgroundColor: '#e8dcc8',
              border: `2px dashed ${colors.薄墨}`
            }}>
              <Camera className="w-16 h-16 mx-auto mb-4" style={{ color: colors.薄墨 }} />
              <p style={{ color: colors.薄墨, fontFamily: "'Noto Serif JP', serif" }}>
                まだアルバムがありません
              </p>
            </div>
          ) : (
            albums.slice().reverse().map(album => (
              <div
                key={album.id}
                className="rounded-lg p-6 transition-all duration-200 hover:shadow-lg active:scale-98"
                style={{ 
                  backgroundColor: '#e8dcc8',
                  border: `2px solid ${colors.黄土}`,
                  boxShadow: '3px 3px 0px rgba(0,0,0,0.1)'
                }}
              >
                <h4 className="text-xl font-bold mb-4" style={{ 
                  color: colors.墨色,
                  fontFamily: "'Noto Serif JP', serif"
                }}>
                  {album.title}
                </h4>
                <div className="grid grid-cols-4 gap-2 mb-3">
                  {album.photos.slice(0, 4).map((photo, index) => (
                    <div key={index} className="aspect-square rounded-lg overflow-hidden border" style={{ borderColor: colors.黄土 }}>
                      <img src={photo} alt="" className="w-full h-full object-cover" />
                    </div>
                  ))}
                </div>
                <div className="flex items-center justify-between text-sm">
                  <span style={{ color: colors.薄墨 }}>
                    {album.photos.length}枚の写真
                  </span>
                  <span style={{ color: colors.薄墨 }}>
                    {new Date(album.createdAt).toLocaleDateString('ja-JP')}
                  </span>
                </div>
              </div>
            ))
          )}
        </div>
      </div>
    </div>
  );

  // 旅の記録ページ
  const RecordsPage = () => (
    <div className="min-h-screen p-8" style={{ backgroundColor: colors.古紙 }}>
      <button 
        onClick={() => setCurrentPage('home')}
        className="mb-6 px-6 py-3 rounded-lg font-bold transition-all duration-300 active:scale-95 hover:scale-110 hover:shadow-lg"
        style={{ 
          backgroundColor: colors.墨色,
          color: colors.古紙,
          fontFamily: "'Noto Serif JP', serif",
          boxShadow: '3px 3px 0px rgba(0,0,0,0.2)'
        }}
      >
        ← 戻る
      </button>

      <div className="max-w-4xl mx-auto">
        <h2 className="text-4xl font-bold mb-8 text-center" style={{ 
          color: colors.萌黄,
          fontFamily: "'Noto Serif JP', serif",
          letterSpacing: '0.1em'
        }}>
          旅の記録
        </h2>

        <div className="grid gap-6">
          {travels.length === 0 ? (
            <div className="text-center py-24 rounded-lg" style={{ 
              backgroundColor: '#e8dcc8',
              border: `2px dashed ${colors.薄墨}`
            }}>
              <Archive className="w-20 h-20 mx-auto mb-6" style={{ color: colors.薄墨 }} />
              <p className="text-xl mb-2" style={{ color: colors.薄墨, fontFamily: "'Noto Serif JP', serif" }}>
                まだ旅の記録がありません
              </p>
              <p style={{ color: colors.薄墨 }}>
                地図ページから旅を始めましょう
              </p>
            </div>
          ) : (
            travels.slice().reverse().map(travel => (
              <div
                key={travel.id}
                className="rounded-lg p-8 transition-all duration-200 hover:shadow-2xl active:scale-98"
                style={{ 
                  backgroundColor: '#e8dcc8',
                  border: `4px solid ${colors.萌黄}`,
                  boxShadow: '4px 4px 0px rgba(0,0,0,0.15)'
                }}
              >
                <div className="flex items-start justify-between mb-6">
                  <div className="flex-1">
                    <h3 className="text-2xl font-bold mb-2" style={{ 
                      color: colors.墨色,
                      fontFamily: "'Noto Serif JP', serif"
                    }}>
                      🎒 {travel.title}
                    </h3>
                    <div className="flex items-center gap-3 text-sm mb-1" style={{ color: colors.薄墨 }}>
                      <span>👤 {travel.userName || '旅人'}</span>
                    </div>
                    <p className="text-sm" style={{ color: colors.薄墨 }}>
                      {new Date(travel.createdAt).toLocaleDateString('ja-JP', {
                        year: 'numeric',
                        month: 'long',
                        day: 'numeric'
                      })}
                    </p>
                  </div>
                  <button
                    onClick={() => shareTravel(travel)}
                    className="px-6 py-3 rounded-lg font-bold transition-all duration-300 active:scale-95 hover:scale-110 hover:shadow-lg"
                    style={{
                      backgroundColor: colors.藍鼠,
                      color: colors.古紙,
                      fontFamily: "'Noto Serif JP', serif",
                      boxShadow: '3px 3px 0px rgba(0,0,0,0.2)'
                    }}
                  >
                    <Share2 className="w-5 h-5" />
                  </button>
                </div>

                <div className="space-y-4">
                  <div className="flex items-center gap-3 p-4 rounded-lg" style={{ backgroundColor: '#f8f5eb' }}>
                    <MapPin className="w-6 h-6" style={{ color: colors.藍鼠 }} />
                    <div>
                      <p className="font-bold" style={{ color: colors.墨色 }}>記録地点</p>
                      <p style={{ color: colors.薄墨 }}>{travel.route.length}ヶ所</p>
                    </div>
                  </div>

                  {travel.diary && (
                    <div className="p-4 rounded-lg" style={{ 
                      backgroundColor: '#f8f5eb',
                      border: `2px solid ${colors.褪朱}`
                    }}>
                      <div className="flex items-center gap-2 mb-2">
                        <BookOpen className="w-5 h-5" style={{ color: colors.褪朱 }} />
                        <p className="font-bold" style={{ color: colors.墨色 }}>
                          {travel.diary.title}
                        </p>
                      </div>
                      <p className="text-sm" style={{ 
                        color: colors.薄墨,
                        lineHeight: '1.8'
                      }}>
                        {travel.diary.content.substring(0, 150)}...
                      </p>
                    </div>
                  )}

                  {travel.album && (
                    <div className="p-4 rounded-lg" style={{ 
                      backgroundColor: '#f8f5eb',
                      border: `2px solid ${colors.黄土}`
                    }}>
                      <div className="flex items-center gap-2 mb-3">
                        <Camera className="w-5 h-5" style={{ color: colors.黄土 }} />
                        <p className="font-bold" style={{ color: colors.墨色 }}>
                          {travel.album.title}
                        </p>
                      </div>
                      <div className="grid grid-cols-4 gap-2">
                        {travel.album.photos.slice(0, 4).map((photo, index) => (
                          <div key={index} className="aspect-square rounded-lg overflow-hidden border" style={{ borderColor: colors.黄土 }}>
                            <img src={photo} alt="" className="w-full h-full object-cover" />
                          </div>
                        ))}
                      </div>
                    </div>
                  )}
                </div>
              </div>
            ))
          )}
        </div>
      </div>
    </div>
  );

  // ユーザー登録モーダル
  const UserModal = () => (
    <div className="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center p-4 z-50 overflow-y-auto">
      <div className="rounded-lg p-8 max-w-2xl w-full my-8" style={{ 
        backgroundColor: colors.古紙,
        border: `4px solid ${colors.渋茶}`,
        boxShadow: '8px 8px 0px rgba(0,0,0,0.3)'
      }}>
        <div className="flex items-center justify-between mb-6">
          <h3 className="text-3xl font-bold" style={{ 
            color: colors.墨色,
            fontFamily: "'Noto Serif JP', serif"
          }}>
            旅人の情報
          </h3>
          <button 
            onClick={() => setShowUserModal(false)}
            className="transition-all duration-200 active:scale-90 hover:scale-110"
          >
            <X className="w-8 h-8" style={{ color: colors.薄墨 }} />
          </button>
        </div>

        <div className="space-y-5">
          {/* 名前 */}
          <div>
            <label className="block mb-2 font-bold" style={{ color: colors.墨色 }}>
              お名前 *
            </label>
            <input
              type="text"
              value={userProfile.name}
              onChange={(e) => setUserProfile({...userProfile, name: e.target.value})}
              placeholder="例: 山田太郎"
              className="w-full px-4 py-3 rounded-lg border-2 focus:outline-none transition-all"
              style={{ 
                borderColor: colors.藍鼠,
                fontFamily: "'Noto Serif JP', serif",
                backgroundColor: '#f8f5eb'
              }}
            />
          </div>

          {/* 年齢 */}
          <div>
            <label className="block mb-2 font-bold" style={{ color: colors.墨色 }}>
              年齢
            </label>
            <input
              type="text"
              value={userProfile.age}
              onChange={(e) => setUserProfile({...userProfile, age: e.target.value})}
              placeholder="例: 30代"
              className="w-full px-4 py-3 rounded-lg border-2 focus:outline-none transition-all"
              style={{ 
                borderColor: colors.藍鼠,
                fontFamily: "'Noto Serif JP', serif",
                backgroundColor: '#f8f5eb'
              }}
            />
          </div>

          {/* 性別 */}
          <div>
            <label className="block mb-2 font-bold" style={{ color: colors.墨色 }}>
              性別
            </label>
            <input
              type="text"
              value={userProfile.gender}
              onChange={(e) => setUserProfile({...userProfile, gender: e.target.value})}
              placeholder="例: 男性、女性、その他"
              className="w-full px-4 py-3 rounded-lg border-2 focus:outline-none transition-all"
              style={{ 
                borderColor: colors.藍鼠,
                fontFamily: "'Noto Serif JP', serif",
                backgroundColor: '#f8f5eb'
              }}
            />
          </div>

          {/* 主な移動手段 */}
          <div>
            <label className="block mb-2 font-bold" style={{ color: colors.墨色 }}>
              主な移動手段（複数選択可）
            </label>
            <div className="grid grid-cols-2 gap-3">
              {['徒歩', '車', 'バイク', '自転車', '公共交通機関'].map(option => (
                <label key={option} className="flex items-center gap-3 p-3 rounded-lg cursor-pointer transition-all hover:shadow-md" style={{ 
                  backgroundColor: userProfile.transportation.includes(option) ? '#e8dcc8' : '#f8f5eb',
                  border: `2px solid ${userProfile.transportation.includes(option) ? colors.褪朱 : colors.薄墨}`
                }}>
                  <input
                    type="checkbox"
                    checked={userProfile.transportation.includes(option)}
                    onChange={() => handleTransportationChange(option)}
                    className="w-5 h-5"
                    style={{ accentColor: colors.褪朱 }}
                  />
                  <span style={{ 
                    color: colors.墨色,
                    fontFamily: "'Noto Serif JP', serif",
                    fontWeight: userProfile.transportation.includes(option) ? 'bold' : 'normal'
                  }}>
                    {option}
                  </span>
                </label>
              ))}
            </div>
          </div>

          {/* 車種 */}
          <div>
            <label className="block mb-2 font-bold" style={{ color: colors.墨色 }}>
              乗っている車、バイク、自転車等の車種
            </label>
            <input
              type="text"
              value={userProfile.vehicle}
              onChange={(e) => setUserProfile({...userProfile, vehicle: e.target.value})}
              placeholder="例: トヨタ・プリウス、ヤマハ・SR400"
              className="w-full px-4 py-3 rounded-lg border-2 focus:outline-none transition-all"
              style={{ 
                borderColor: colors.藍鼠,
                fontFamily: "'Noto Serif JP', serif",
                backgroundColor: '#f8f5eb'
              }}
            />
          </div>

          {/* カメラ機種 */}
          <div>
            <label className="block mb-2 font-bold" style={{ color: colors.墨色 }}>
              カメラの機種名
            </label>
            <input
              type="text"
              value={userProfile.camera}
              onChange={(e) => setUserProfile({...userProfile, camera: e.target.value})}
              placeholder="例: Canon EOS R5、iPhone 15 Pro"
              className="w-full px-4 py-3 rounded-lg border-2 focus:outline-none transition-all"
              style={{ 
                borderColor: colors.藍鼠,
                fontFamily: "'Noto Serif JP', serif",
                backgroundColor: '#f8f5eb'
              }}
            />
          </div>

          {/* おすすめの旅先 */}
          <div>
            <label className="block mb-2 font-bold" style={{ color: colors.墨色 }}>
              おすすめの旅先
            </label>
            <textarea
              value={userProfile.recommendations}
              onChange={(e) => setUserProfile({...userProfile, recommendations: e.target.value})}
              placeholder="例: 京都の嵐山、北海道の美瑛など"
              rows="3"
              className="w-full px-4 py-3 rounded-lg border-2 focus:outline-none resize-none transition-all"
              style={{ 
                borderColor: colors.藍鼠,
                fontFamily: "'Noto Serif JP', serif",
                lineHeight: '1.8',
                backgroundColor: '#f8f5eb'
              }}
            />
          </div>

          <button
            onClick={saveUserProfile}
            className="w-full px-8 py-4 rounded-lg text-xl font-bold transition-all duration-300 active:scale-95 hover:scale-110 hover:shadow-2xl"
            style={{
              backgroundColor: colors.萌黄,
              color: colors.古紙,
              fontFamily: "'Noto Serif JP', serif",
              boxShadow: '4px 4px 0px rgba(0,0,0,0.2)'
            }}
          >
            保存する
          </button>
        </div>
      </div>
    </div>
  );

  return (
    <div style={{ 
      fontFamily: "'Noto Sans JP', sans-serif",
      backgroundColor: colors.古紙,
      minHeight: '100vh'
    }}>
      <link href="https://fonts.googleapis.com/css2?family=Noto+Serif+JP:wght@400;700;900&family=Noto+Sans+JP:wght@400;700&display=swap" rel="stylesheet" />
      
      {currentPage === 'home' && <HomePage />}
      {currentPage === 'map' && <MapPage />}
      {currentPage === 'diary' && <DiaryPage />}
      {currentPage === 'photos' && <PhotosPage />}
      {currentPage === 'records' && <RecordsPage />}
      {showUserModal && <UserModal />}
    </div>
  );
}
