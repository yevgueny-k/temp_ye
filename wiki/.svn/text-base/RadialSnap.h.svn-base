#ifndef RADIALSNAP_H
#define RADIALSNAP_H

#include <QEIDesignDocument.h>
#include <QEIDesignSnapAdapter.h>
#include <QEIExtension.h>

class RadialSnapAdapter: public QEIDesignSnapAdapter
{
public:
  RadialSnapAdapter(QEIDesignDocument* d); // d != 0

  // -- QEIInterface

  virtual QString uid() const;
  virtual void restoreSettings(const QSettings& settings, unsigned int context);
  virtual void saveSettings(QSettings& settings, unsigned int context) const;

  // -- QEIDesignSnapAdapter

  virtual QSharedPointer<QEIDesignSnapPara> snapDefaultPara() const;
  virtual void snap(QSharedPointer<QEIDesignSnapDescriptor> context);
  virtual QList<QEUpdatingRect> snapGetPaintAreas(QSharedPointer<QEIDesignSnapDescriptor> context, const QList<QEIView*>& destViews);
  virtual void snapPaint(QSharedPointer<QEIDesignSnapDescriptor> context, QPainter* painter, const QRectF& rect);

  // -- Custom

  QEIDesignDocument* document() const { return mDocument; }

private:
  QEIDesignDocument*   mDocument;
};

struct RadialSnapPara: public QEIDesignSnapPara
{
public:

  // -- QEIInterface

  virtual QString uid() const;
  virtual void restoreSettings(const QSettings& settings, unsigned int context);
  virtual void saveSettings(QSettings& settings, unsigned int context) const;

  // -- QEIDesignSnapAdapter

  virtual QEDesignSnapAdapterType snapType() const { return kUserDesignSnapAdapter; }
  virtual QString snapTypeName() const { return "Radial Snap"; }
  virtual bool isOn() const { return mIsOn; }
  virtual void setIsOn(bool x) { mIsOn = x; }
  virtual QSharedPointer<QEIDesignSnapPara> clone() const;

  // -- Custom

  double paramAngleStep() const { return mParamAngleStep; } // in degrees, min 0.001
  QPointF paramRaySourcePoint() const { return mParamRaySourcePoint; }

  void setParamAngleStep(double x);
  void setParamRaySourcePoint(const QPointF& x);

  RadialSnapPara(QEIDesignDocument* d, bool isOn); // d != 0

  QEIDesignDocument* document() const { return mDocument; }

private:
  bool mIsOn;
  QEIDesignDocument* mDocument;

  double mParamAngleStep;
  QPointF mParamRaySourcePoint;
};

#endif // RADIALSNAP_H
